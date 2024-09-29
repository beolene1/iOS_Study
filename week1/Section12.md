# Section 12 - Web API and MV Pattern 

* Json data를 디코드하기 위한 방식은 UIKit과 동일. 



* 폴더를 
Models, 
Clients, 
Utilities, 
Extension으로 나눈 구조가 흥미롭다.

<br/>

```swift
enum APIEndpoint {
    
    static let baseURL = "https://api.openweathermap.org"
    
    case coordinatesByLocationName(String)
    case weatherByLatLon(Double, Double)
    
    private var path: String {
        switch self {
            case .coordinatesByLocationName(let city):
                return "/geo/1.0/direct?q=\(city)&appid=\(Constants.Keys.weatherAPIKey)"
            case .weatherByLatLon(let lat, let lon):
                return "/data/2.5/weather?lat=\(lat)&lon=\(lon)&appid=\(Constants.Keys.weatherAPIKey)"
        }
    }
    
    static func endpointURL(for endpoint: APIEndpoint) -> URL {
        let endpointPath = endpoint.path
        return URL(string: baseURL + endpointPath)!
    }
    
}
```

* 여러 개의 API call을 할 때 API Endpoint만 모아놓는 것
* 내 고유의 API Key는 Constants 파일에 따로 저장한 것

<br/><br/><br/>
```swift
struct ContentView: View {
    
    @State private var city: String = ""
    @State private var isFetchingWeather: Bool = false
    
    let geocodingClient = GeocodingClient()
    let weatherClient = WeatherClient()
```
    

* State로 저장한 이유는 화면에 변경사항을 바로바로 반영해야 하기 때문에

* 각각의 client는 api call을 통해 받아온 정보를 디코드까지 해서 예쁘게 가져오는 역할
<br/><br/>

```swift
@State private var weather: Weather?
```


얘가 있으면 화면에 온도를 띄우고 없으면 빈 화면
<br/>

```swift
    private func fetchWeather() async {
        do {
            guard let location = try await geocodingClient.coordinateByCity(city) else { return }
            weather = try await weatherClient.fetchWeather(location: location) // 옵셔널 값인 웨더에 값을 저장해준다. 
        } catch {
            print(error)
        }
    }
```


* Do- catch 문으로 안전하게

* Try-await 문으로 안전하게

<br/><br/>
사용자의 현재 위도경도를 가져오고 그걸 기반으로 현재 위치의 날씨 정보를 가져오는 함수

```swift
    var body: some View {
        VStack {
            TextField("City", text: $city)
                .textFieldStyle(.roundedBorder)
                .onSubmit {
                    isFetchingWeather = true
                }.task(id: isFetchingWeather) {
                    if isFetchingWeather {
                        await fetchWe/Users/beolene/Desktop/iOSPlusStudy/week1/Section12.txtather() // 여기서 함수 호출
                        isFetchingWeather = false 
                        city = "" // 검색했던 부분 초기화
                    }
                }
            
            Spacer()
            if let weather { // API response가 정상적으로 도착하여 값이 존재하게 되었을 때
                Text(MeasurementFormatter.temperature(value: weather.temp))
                    .font(.system(size: 100))
            }
            Spacer()
        }
        .padding()
    }
```

서치 바에 검색한 도시의 이름으로 API call을 한 뒤 화면에 보여준다. 
