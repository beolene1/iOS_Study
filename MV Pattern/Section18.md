# Section 18

```swift
import Foundation
import CoreData

@objc(BudgetCategory)
public class BudgetCategory: NSManagedObject {
    
    public override func awakeFromInsert() { // budget category를 하나 만들 때마다 실행됨
        self.dateCreated = Date() 
    }
    
}
```

```swift
import Foundation
import CoreData

class CoreDataManager {
    
    // singleton, coreDataManager에 접근할 유일한 방법
    static let shared = CoreDataManager()  
    private var persistentContainer: NSPersistentContainer
    
    private init() {
        persistentContainer = NSPersistentContainer(name: "BudgetModel")
        persistentContainer.loadPersistentStores { description, error in
            if let error {
                fatalError("Unable to initialize Core Data stack \(error)")
            }
        }
    }
    
    var viewContext: NSManagedObjectContext {
        persistentContainer.viewContext 
    }
}
```

- CoreData - data model 파일이 따로 있는 게 신기. 코드 형식이 아니라 info.plist같이 생겼다.

```swift

@Environment(\.managedObjectContext) private var viewContext
//BudgetsAppApp 파일에서 ContentView에 inject 해줬기 때문에 자식 뷰들에서 사용가능

private func save() {
        
        let budgetCategory = BudgetCategory(context: viewContext)
        budgetCategory.title = title
        budgetCategory.total = total
        
        // save the context
        do {
            try viewContext.save() // context의 save 메소드
            dismiss()
        } catch {
            print(error.localizedDescription)
        }
        
    }
```

### Core Data

**: 앱의 모델 계층**이자, 데이터베이스, ORM 등의 기능을 가진 **객체 그래프 관리 프레임워크**

- 객체 그래프: 메모리에 있는 객체들과 그 객체들간의 관계 (수많은 객체들이 메모리에서 관계를 형성하고 있는데, 그 형태 그대로 디스크에 저장할 필요가 있고 Core Data는 그 과정을 지원한다)
- Database: Core Data의 일부 기능 중 하나
- ORM: 관계형 데이터베이스와 소통하기 위해서는 SQL이란 언어가 필요한데, ORM은 내가 앱을 만드는 데 사용하고 있는 언어를 SQL문으로 바꿔준다.

앱에서 사용하는 데이터 중 로컬 디바이스에 저장해놓고 사용하는 데이터들의 모델은 코어데이터로 관리한다고 보면 된다.

### Core Data Model 만들기

1. 프로젝트에 Core Data 추가하기
2. Entity 구성하기 (e.g. BudgetCategory)
3. Attributes 구성하기 - Attributes는 Entity의 property이다. (e.g. dateCreated, title, total)
4. 관계 구성하기 - Entity가 여러 개일 경우 관계를 설정해줄 수 있음
5. 코드 생성 - Entity의 instance를 만드는데 사용할 class를 생성해줘야 한다 (e.g. public class BudgetCategory)