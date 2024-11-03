# Section 20

Transaction 삭제하기

- .contentShape(Rectangle()) :직사각형이든 원이든, 도형 전체를 클릭
- .onLongPressGesture {} : 길게 눌렀을 때

```swift
struct ContentView: View {
    
    @Environment(\.managedObjectContext) private var viewContext
   // @FetchRequest(sortDescriptors: []) private var budgetCategoryResults: FetchedResults<BudgetCategory>
    @FetchRequest(fetchRequest: BudgetCategory.all) var budgetCategoryResults
    @State private var sheetAction: SheetAction?
    
    var total: Double {
        budgetCategoryResults.reduce(0) { result, budgetCategory in
            return result + budgetCategory.total
        }
    }
    
    private func deleteBudgetCategory(budgetCategory: BudgetCategory) {
        
        viewContext.delete(budgetCategory)
        do {
            try viewContext.save()
        } catch {
            print(error)
        }
    }
    
    private func editBudgetCategory(budgetCategory: BudgetCategory) {
        sheetAction = .edit(budgetCategory)
    }
    
    var body: some View {
        NavigationStack {
            VStack {
                
                HStack {
                    Text("Total Budget - ")
                    Text(total as NSNumber, formatter: NumberFormatter.currency)
                        .fontWeight(.bold)
                }
                
                // 가져오기, 삭제하기, 편집하기 함수를 다 넘겨주기
                BudgetListView(budgetCategoryResults: budgetCategoryResults, onDeleteBudgetCategory: deleteBudgetCategory, onEditBudgetCategory: editBudgetCategory)
            }
            .sheet(item: $sheetAction, content: { sheetAction in
                // display the sheet
                switch sheetAction {
                    case .add:
                        AddBudgetCategoryView()
                    case .edit(let budgetCategory):
                        AddBudgetCategoryView(budgetCategory: budgetCategory)
                }
            })
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Add Category") {
                        sheetAction = .add
                    }
                }
            }.padding()
        }
       
    }
}

```

**Form**

: 

- HStack, VStack 같은 컨테이너와 비슷하게 작동 (이 안에 여러 뷰를 넣을 수 있음)
- 특정 컨트롤 요소에 대해 더 보기 좋게 잘 작동하게 해준다.
- 폼 안에서 섹션별로 구분지어줄 수도 있다.
- 특정 뷰를 조건에 따라 활성화/비활성화 가능

```swift
enum SheetAction: Identifiable {
    
    case add
    case edit(BudgetCategory)
    
    var id: Int {
        switch self {
            case .add:
                return 1
            case .edit(_):
                return 2
        }
    }
    
}
```

enum으로 관리