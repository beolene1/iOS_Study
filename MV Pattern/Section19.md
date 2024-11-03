# Section 19

Transaction Core Data Model 만들기

dataModel 파일에서 Relationship 추가해주기 - transaction

```swift
import Foundation
import CoreData

@objc(Transaction)
public class Transaction: NSManagedObject {
    
    public override func awakeFromInsert() {
        self.dateCreated = Date()
    }
}
```

위의 transaction 파일?을 따로 만들어주지 않으면 에러가 난다.

- transaction의 owner는 budgetcategory 하나, budgetcategory에는 여러 개의 transaction이 붙어있을 수 있음 - relationship 설정에서 주의해야

### NSFetchRequest

: SwiftUI에서 지원하는 Core Data 가져오기 요청 작업을 위한 전용 property wrapper(swift에서는 다른 방법을 사용)

### NSManagedObjectContext

:  

- 대부분의 작업은 context에서 처리되고 managed object는 항상 context안에 존재한다.
- 등록된 managed object는 context를 통해 persistent store에 저장되고, 이미 저장되어 있는 Managed Object를 수정하고 저장 할 때도 Managed Object를 복사하여 Context에 등록해야 한다.
Managed Object를 사용하기 위해선 반드시 Context에 등록이 필요하다.
- 데이터를 변경한 후 context에서 save()를 호출해야 데이터가 저장된다.

```swift
struct TransactionListView: View {
    
    @FetchRequest var transactions: FetchedResults<Transaction>
    
    init(request: NSFetchRequest<Transaction>) {
        _transactions = FetchRequest(fetchRequest: request)
    }
    
    var body: some View {
        if transactions.isEmpty {
            Text("No transactions.")
        } else {
            List {
                ForEach(transactions) { transaction in
                    HStack {
                        Text(transaction.title ?? "")
                        Spacer()
                        Text(transaction.total as NSNumber, formatter: NumberFormatter.currency)
                    }
                }
            }
        }
    }
}

```

**BudgetDetailView**

```swift
private func saveTransaction() {
        
        do {
            let transaction = Transaction(context: viewContext)
            transaction.title = title
            transaction.total = Double(total)!
            
            budgetCategory.addToTransactions(transaction)
            try viewContext.save()
        } catch {
            print(error)
        }
        
    }
```