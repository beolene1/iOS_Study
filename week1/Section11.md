# Section 11 - Understanding State and Binding

### @State

* UI는 ‘state’에 있는 데이터를 반영한다.

* struct의 프로퍼티는 mutable하지 않기 때문에 값을 바꿀 수 없다. Mutating func로 지정해도 값을 바꾸진 못한다. 값을 바꾸기 위해서는 State라는 property wrapper로 묶어줘야 한다.

```swift
@State private var count: Int = 0
```
<br/>

이렇게 생겼다.   

* State의 상태가 변할 때마다 body는 re-evaluated 된다.
State는 UI가 데이터를 실시간으로 반영할 수 있게 도와준다. 


### @Binding

* "$" < 요 달러 사인으로 표시된다. State로 정의된 변수?를 Binding으로 넘겨줄 수 있다.

***
<br/>

* Textfield의 modifier .onSubmit {} < 엔터키를 눌렀을 때의 행동을 정의



```swift
List(friends, id: \.self { friend in 
(중략)
}
```

<br/>

여기서 friends가 내가 만든 type이 아니기 때문에 Identifiable에 conform하게 만드는 대신 id: \.self를 써주는 것.   


* NavigationStack을 계속 가져가고 싶으면 앱 파일에서 ContentView를 감싸야지, 프리뷰에서 감싸면 프리뷰에만 적용되는 것이다. 

* list에 .searchable(text: )를 더해줄 수 있다. 매우 간단하게. 네비게이션 스택이 필요하고, parameter로는 String의 Binding을 넘겨줘야 한다. 

* frameㅇ에 maxHeight, maxWidth 있는데, .infinity로 설정하면 화면 꽉 차는 크기로 키울 수 있다.


*** 

<br/>

ContentView 안에서 State로 선언된 isLightOn를 바꾸고 싶으면 어떻게 해야 하나    
-> ContentView 안의 LightbulbView에서 Binding으로 isOn 선언, ContentView에서는 LightbulbView에 $isLightOn을 파라미터로(isOn으로) 넘겨줘야 한다.  

<br/>  

```swift
import SwiftUI

struct LightBulbView: View {
    
    @Binding var isOn: Bool
    
    var body: some View {
        VStack {
            Image(systemName: isOn ? "lightbulb.fill": "lightbulb")
                .font(.largeTitle)
                .foregroundStyle(isOn ? .yellow: .black)
            Button("Toggle") {
                isOn.toggle()
            }
        }
    }
}

struct ContentView: View {
    
    @State private var isLightOn: Bool = false
    
    var body: some View {
        VStack {
            LightBulbView(isOn: $isLightOn)
        }
        .padding()
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        .background(isLightOn ? .black: .white)
    }
}
```
<br/>

* 화면의 background color는 ContentView에서, 화면 속 이미지는 LightbulbView에서 정의하고 있는데, isLightOn이 바뀔 때마다 따로 정의된 이 둘이 동시에 변화한다. 

* Child view가 parent view와 소통-연결되길 원할 때 binding을 쓴다. 


### Environment Object

* 스유에서 global state는 environment object를 이용하여 만들어진다. 
뷰가 막 다섯 개씩 있어서 피라미드 구조를 형성한다고 생각해 보자. 이걸 일일이 다 Binding으로 넘겨주기엔 너무 복잡해진다. 

<br/>

```swift
// Pre iOS 17
class AppState: ObservableObject {
    @Published var isOn: Bool = false
}
```
프리뷰에서 우리의 앱 state를 root object(여기서는 ContentView?)에 주입해야 한다. 그렇게 해야 contentView의 자식 뷰들이 environment object를 인식할 수 있다.
<br/>


```swift
#Preview {
    ContentView()
        .environmentObject(AppState())
}
```
이건 프리뷰에서만을 위한 조치이고, 전체 앱을 위해서는 다른 방식을 사용해야 한다. 
<br/>


```swift
    @StateObject private var appState = AppState()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
        }
    }
```
앱 파일에서 주입..inject 해준다. 
둘 모두 같은 기능을 한다. 

<br/>

이렇게 하면 ContentView에서 LightBulbView에 직접 데이터를 전달하지 않고도 LightBulbView가 직접 데이터에 접근할 수 있게 된다.
<br/>

```swift
struct LightBulbView: View {
    
    @EnvironmentObject private var appState: AppState
    
    var body: some View {
	(중략)
     }
}
```

* Binding은 1-2 level 깊이에 적합하다. 
하지만 4-5 level이 된다면 그땐 Environment object를 쓰는 걸 권장

<br/><br/>


### iOS 17 이후

Pre

```swift
class AppState: ObservableObject {
    @Published var isOn: Bool = false
}
```

였는데, 

```swift
@Observable
class AppState {
    var isOn: Bool = false
}
```

가 됐다. 
<br/>
@Observable이라는 키워드를 쓰는 순간 모든 프로퍼티들은 자동으로 publish되기 때문. 
그리고 Observable은 반드시 class와 함께 사용해야 한다. 

프리 
```swift
 @EnvironmentObject private var appState: AppState
```

얘도 
```swift
@Environment(AppState.self) private var appState: AppState
```

이게 됐다.


앱 파일에서 
```swift
    @StateObject private var appState = AppState()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
        }
    }
```

라고 써줬던 걸 
```swift
@State private var appState = AppState()
```
라고 바꾼다. 

