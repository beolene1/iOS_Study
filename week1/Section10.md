# Section 10 - Building List and Navigation

* 동적 리스트를 화면에 표시하는 법을 배운다.
> 동적 리스트란? 항목이 추가, 변경, 삭제될 수 있고, 그 변화를 동적으로 반영하는 리스트가 동적 리스트이다.<br/>

* 내가 만든 Emotion이라는 type의 array를 ContentView 안에 선언해주자.    
-> 이 array를 뷰에서 List로 표현하기 위해서는 이 array의 type이 Identifiable에 conform해야 한다.    

> Identifiable이란 무엇일까?     
> Identifiable이란 프로토콜이다. 고유 식별자를 가진 type을 위한 프로토콜이다. 이 프로토콜을 채택한 type은 고유한 식별자(unique identifier)를 제공해야 한다. **list에서 각 항목을 고유하게 식별할 수 있는 id라는 프로퍼티가 객체에 존재해야 한다.**.<br/>   

요 id는 아주 유니크해야 하기 때문에 **UUID로 저장**한다.<br/><br/>

#### 여기서 주의할 점

```swift
 let id : UUID
```

가 아니라

```swift
 let id = UUID()
```

라고 써줘야 한다. 


```swift
struct Emotion: Identifiable {
    let id = UUID()
    let name: String
    let image: String
    let scores: Int
}
```<br/><br/><br/>

#### UIKit에서 tableview를 화면에 띄우기 위해 필요한 코드량보다 훨씬 적은 코드로 리스트 구현이 가능하다.


* Text에다가 커서 갖다대고 오른쪽 버튼 누르면 Embed in HStack, VStack, 같은애들이 나온다. 직접 괄호 안에 넣어가며 쓰는 것보다 이런 기능을 이용하는 게 훨씬 빠르고 실수도 없을 것.<br/><br/>


### ContentView

```swift
 import SwiftUI

struct ContentView: View {
    
    let emotions = [
        Emotion(name: "shyness", image: "shy", scores: 20),
        Emotion(name: "embarassment", image: "embarrassed", scores: 10),
        Emotion(name: "anticipation", image: "starryEyes", scores: 100)
    ]
    
    var body: some View {
        List(emotions) { emotion in
            HStack(alignment: .top) {	// 여기 HStack 부분이 cell과 같은 역할을 하는 중
                Image(emotion.image)
                    .resizable()
                    .aspectRatio(contentMode: .fit)
                    .frame(width: 100, height: 100)
                VStack(alignment: .leading) {
                    Text(emotion.name)
                        .fontWeight(.bold)
                        .font(.title3)
                    Text("\(emotion.scores)점")
                        .foregroundStyle(.gray)
                        .fontWeight(.bold)
                }
            }
        }
    }
}
```

<img width="258" alt="shyness" src="https://github.com/user-attachments/assets/1e3d158f-d572-4b4e-b354-ad973d813ac3"><br/><br/><br/>

***


* 코드 속 HStack은 화면 속 리스트의 cell 하나와 같은 역할을 하는데, HStack에서 우클릭하면 Extract Subview라는 항목이 있다. 코드를 더 깔끔하게 분리해줄 수 있다.    

-> 기존에 ContentView 안에서 정의되었던 emotions array 때문에 에러가 나는데, 그래서 extract된 subview에다 emotions을 전달해줘야 한다. (아래 참고)<br/>       



```swift
 var body: some View {
        List(emotions) { emotion in
            EmotionCellView(emotion: emotion)
        }
    }
```<br/><br/>


그리고 extracted된 cell 안에서 받아온 emotion을 저장할 프로퍼티를 선언?해줘야 한다.    

```swift
 struct EmotionCellView: View {
    
    let emotion: Emotion
    
    var body: some View {
        HStack(alignment: .top) {
            Image(emotion.image)
                .resizable()
~~ 후략
```<br/><br/><br/>


### Navigation 더하기

#### 1. Navigation Stack 필요.  
: Navigation Controller와 유사한 역할을 한다.

#### 2. Navigation Link 필요.   

**navigationStack(List(navigationLink(EmotionCellDetailView)))**       

* navigationLink에 전달하는 값(Emotion type)은 Hashable 프로토콜을 준수해야 한다. 

* 각각의 cell에 > 표시가 생긴다.

#### 3. 도착 지점을 .navigationDestination으로 표현. 이 클로저 내에서 각각 뷰를 리턴한다.<br/><br/>   



```swift
     var body: some View {
        NavigationStack {
            List(emotions) { emotion in
                NavigationLink(value: emotion) {
                    EmotionCellView(emotion: emotion)
                }
                
            }.navigationTitle("Emotions")
            .navigationDestination(for: Emotion.self) { emotion in
                    Image(emotion.image)
            }
        }
    }
```<br/>


* 여기서 navigationDestination(for: Emotion.self) 안하고 Int.self를 parameter로 넘겨줬으면, 아예 cell에 클릭 자체가 불가능하다. 이는 네비게이션 링크의 value가 Emotion type이기 때문에 destination도 똑같은 type으로? 만들어줘야 하기 때문이다.<br/>


> navigationDestination은 조금 높은 레벨에서 정의해야 한다?<br/>

* 타이틀 글자가 너무 길어서 잘리면 어떻게 해야 하는가?     
-> 
```swift  
.navigationBarTitleDisplayMode(.inline)
```<br/><br/>


### .onTapGesture

* 이미지 클릭 시의 반응 정하기
* 삼항연산자를 통해 zoomed의 값에 따라 이미지가 다르게 보이도록 설정할 수 있다.
* 애니메이션도 넣을 수 있음. withAnimation(앞에 온점 없음) 블록으로 toggle() 코드 감싸주기.<br/><br/><br/>


### EmotionDetailScreen


```swift
 struct EmotionDetailScreen: View {
    
    let emotion: Emotion //여기서 이걸 명시해줘야 ContentView의 navigationDestination에서 emotion을 여기로 넘겨줄 수 있음
    @State private var zoomed: Bool = false
    
    
    var body: some View {
        VStack {
            Spacer()
            Image(emotion.image)
                .resizable()
                .aspectRatio(contentMode:.fit)
                .frame(width: zoomed ? 800: 300, height: zoomed ? 800: 300)
                .onTapGesture {
                    withAnimation {
                        zoomed.toggle()
                    }
                }
            Text(emotion.name)
                .font(.title)
            Text("\(emotion.scores)점")
            Spacer()
        }.navigationTitle(emotion.name)
        //.navigationBarTitleDisplayMode(.inline)
    }
}

#Preview {
    NavigationStack {
        EmotionDetailScreen(emotion: Emotion(name: "shy", image: "shy", scores: 20))
    }
}
```

### ContentView


```swift
 struct ContentView: View {
    
    let emotions = [
        Emotion(name: "shyness", image: "shy", scores: 20),
        Emotion(name: "embarassment", image: "embarrassed", scores: 10),
        Emotion(name: "anticipation", image: "starryEyes", scores: 100)
    ]
    
    var body: some View {
        NavigationStack {
            List(emotions) { emotion in
                NavigationLink(value: emotion) {
                    EmotionCellView(emotion: emotion)
                }
                
            }.navigationTitle("Emotions")
            .navigationDestination(for: Emotion.self) { emotion in
                EmotionDetailScreen(emotion: emotion)
            }
        }
    }
    
}
```

<img width="250" alt="스크린샷 2024-09-29 오전 1 16 12" src="https://github.com/user-attachments/assets/b89343ac-f244-42db-bdee-f8974ead55af">

<img width="262" alt="스크린샷 2024-09-29 오전 1 16 44" src="https://github.com/user-attachments/assets/e3ffbb89-9e96-4b24-861b-5e01b6326c94">
