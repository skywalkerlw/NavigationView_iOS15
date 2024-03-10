# NavigationView_iOS15
iOS 15 下 NavigationView和NavigationLink的各种用法

## 当`NavigationLink`的`isActive`为optional时

```swift
public extension Binding where Value == Bool {
    init<Wrapped>(bindingOptional: Binding<Wrapped?>) {
        self.init(
            get: {
                bindingOptional.wrappedValue != nil
            },
            set: { newValue in
                guard newValue == false else { return }

                /// We only handle `false` booleans to set our optional to `nil`
                /// as we can't handle `true` for restoring the previous value.
                bindingOptional.wrappedValue = nil
            }
        )
    }
}

extension Binding {
    public func mappedToBool<Wrapped>() -> Binding<Bool> where Value == Wrapped? {
        return Binding<Bool>(bindingOptional: self)
    }
}

struct NavigationStackModifier<Item, Destination: View>: ViewModifier {
    let item: Binding<Item?>
    let destination: (Item) -> Destination

    func body(content: Content) -> some View {
        content.background(NavigationLink(isActive: item.mappedToBool()) {
            if let item = item.wrappedValue {
                destination(item)
            } else {
                EmptyView()
            }
        } label: {
            EmptyView()
        })
    }
}

public extension View {
    func navigationDestination<Item, Destination: View>(
        for binding: Binding<Item?>,
        @ViewBuilder destination: @escaping (Item) -> Destination
    ) -> some View {
        self.modifier(NavigationStackModifier(item: binding, destination: destination))
    }
}
```

## `NavigationLink`点击后执行一个side effect动作

https://stackoverflow.com/questions/59870199/swiftui-how-to-do-additional-work-when-clicking-a-navigationlink

```swift
struct ContentView: View {
    @State private var navIsActive = false
    
    var body: some View {
        HStack {
            Spacer()
            
            NavigationLink(isActive: $navIsActive) {
                Success()
                    .navigationBarBackButtonHidden(true)
            } label: {
                Text("Finish")
            }
            
            Spacer()
        }
        .onChange(of: navIsActive) { newValue in
            if newValue {
                print("Sending to Storage")
            }
        }
    }
}
```
