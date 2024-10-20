
# Design Patterns in Golang

Design patterns in Golang can help you write cleaner, more maintainable, and scalable code. This repository demonstrates several common design patterns using Go.

## Table of Contents

1. [Singleton](#1-Singletonsingleton)
2. [Factory Method](#factory-method)
3. [Abstract Factory Pattern](#Abstract-Factory-Pattern)
4. [Builder Pattern](#Builder-Pattern)
5. [Prototype Pattern](#Prototype-Pattern)
6. [Observer](#observer)
7. [Strategy](#strategy)
8. [Decorator](#decorator)
9. [Command](#command)
10. [Adapter](#adapter)
11. [Conclusion](#conclusion)

---

## 1. Singleton

The Singleton pattern Ensure that a class has only one instance and provides a global access point to it.

This pattern is useful when you need a single instance for components like a configuration manager or a database connection throughout your application.

```go
package main

import (
 "fmt"
 "sync"
)

type (
 singleton struct {
  data string
 }
)

var instance *singleton
var once sync.Once

// Function to return the single instance
func GetInstance() *singleton {
 // Use sync.Once to ensure the instance is created only once
 once.Do(func() {
  instance = &singleton{data: "This is a singleton"}
 })

 return instance
}

func main() {
 s1 := GetInstance()
 s2 := GetInstance()

 // Both should point to the same instance
 fmt.Println(s1 == s2) // true
 fmt.Println(s1.data)  // "This is a singleton"
}
```
- sync.Once ensures that the instance is only created once, even in the case of concurrent calls to GetInstance.
- The Singleton pattern prevents multiple instances from being created, which can be crucial for managing system-wide resources.

---

## 2. Factory Method

The Factory Method pattern Define an interface for creating an object, but let subclasses alter the type of objects that will be created.

It Provides a way to delegate the instantiation logic to child classes, allowing flexibility in object creation.

This pattern is useful when you have multiple types of similar objects, like animals (dog, cat) or vehicles (car, bike), and you want to abstract away the creation process.

```go
package main

import (
 "fmt"
)

type (
 Animal interface {
  Speak() string
 }

 Dog struct{}
 Cat struct{}
)

func (d Dog) Speak() string {
 return "Woof!"
}

func (c Cat) Speak() string {
 return "Meow!"
}

func AnimalFactory(animalType string) Animal {
 if animalType == "dog" {
  return &Dog{}
 } else if animalType == "cat" {
  return &Cat{}
 }
 return nil
}

func main() {
 dog := AnimalFactory("dog")
 fmt.Println(dog.Speak()) // Woof!

 cat := AnimalFactory("cat")
 fmt.Println(cat.Speak()) // Meow!
}
```
---
## 3. Abstract Factory Pattern

Abstract Factory provides an interface for creating families of related or dependent objects without specifying their concrete classes. In Go, it can be implemented by defining different factory interfaces.

It is Useful when the system needs to be independent of how its objects are created, composed, and represented. It allows the creation of families of related objects.

```go
package main

import (
 "fmt"
)

// Define an abstract factory
type (
 GUIFactory interface {
  CreateButton() Button
  CreateCheckbox() Checkbox
 }
)

// Define interfaces for products
type (
 Button interface {
  Press() string
 }

 Checkbox interface {
  Check() string
 }

 // Implement concrete products for Windows
 WindowsButton   struct{}
 WindowsCheckbox struct{}

 // Implement concrete products for Mac
 MacButton   struct{}
 MacCheckbox struct{}

 // Implement factories for each platform
 WindowsFactory struct{}
 MacFactory     struct{}
)

func (w *WindowsButton) Press() string { return "Windows Button Pressed" }

func (w *WindowsCheckbox) Check() string { return "Windows Checkbox Checked" }

func (m *MacButton) Press() string { return "Mac Button Pressed" }

func (m *MacCheckbox) Check() string { return "Mac Checkbox Checked" }

func (w *WindowsFactory) CreateButton() Button     { return &WindowsButton{} }
func (w *WindowsFactory) CreateCheckbox() Checkbox { return &WindowsCheckbox{} }

func (m *MacFactory) CreateButton() Button     { return &MacButton{} }
func (m *MacFactory) CreateCheckbox() Checkbox { return &MacCheckbox{} }

func main() {
 // Get a Windows factory
 var wf GUIFactory = &WindowsFactory{}
 button := wf.CreateButton()
 checkbox := wf.CreateCheckbox()

 fmt.Println(button.Press())   // Output: Windows Button Pressed
 fmt.Println(checkbox.Check()) // Output: Windows Checkbox Checked

 var mf GUIFactory = &MacFactory{}
 button = mf.CreateButton()
 checkbox = mf.CreateCheckbox()

 fmt.Println(button.Press())   // Output: Mac Button Pressed
 fmt.Println(checkbox.Check()) // Output: Mac Checkbox Checked
}

```
---
## 4. Builder Pattern

The Builder pattern separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

Complex objects are often constructed step by step. The Builder pattern provides a flexible solution for creating such objects, breaking down the instantiation process.

This pattern is useful when constructing objects like cars, houses, or reports where the process involves many steps and configurations.

```go
package main

import (
 "fmt"
)

// Product to be built
type (
 House struct {
  windows string
  doors   string
  roof    string
 }

 // Concrete builder for a villa
 VillaBuilder struct {
  house House
 }

 // Director controls the building process
 Director struct {
  builder HouseBuilder
 }
)

// Builder interface
type (
 HouseBuilder interface {
  SetWindows() HouseBuilder
  SetDoors() HouseBuilder
  SetRoof() HouseBuilder
  Build() *House
 }
)

func (v *VillaBuilder) SetWindows() HouseBuilder {
 v.house.windows = "Villa Windows"
 return v
}

func (v *VillaBuilder) SetDoors() HouseBuilder {
 v.house.doors = "Villa Doors"
 return v
}

func (v *VillaBuilder) SetRoof() HouseBuilder {
 v.house.roof = "Villa Roof"
 return v
}

func (v *VillaBuilder) Build() *House {
 return &v.house
}

func (d *Director) Construct() *House {
 return d.builder.SetWindows().SetDoors().SetRoof().Build()
}

func main() {
 director := &Director{}

 // Build a villa
 v_builder := &VillaBuilder{}
 director.builder = v_builder
 villa := director.Construct()
 fmt.Println(*villa) // Output: {Villa Windows Villa Doors Villa Roof}
}
```
---
## 5. Prototype Pattern

The Prototype pattern allows for creating new objects by copying an existing object (prototype) rather than creating from scratch

When the cost of creating a new object is high and there is an existing object that can be cloned to reuse, the Prototype pattern is useful.

```go
package main

import (
 "fmt"
)

// Cloneable interface
type (
 Cloneable interface {
  Clone() Cloneable
 }
)

// Concrete struct (prototype)
type (
 Product struct {
  name     string
  category string
 }
)

// Clone method creates a copy of the Product
func (p *Product) Clone() Cloneable {
 return &Product{name: p.name, category: p.category}
}

func (p *Product) SetName(name string) {
 p.name = name
}

func (p *Product) GetDetails() string {
 return fmt.Sprintf("Product Name: %s, Category: %s", p.name, p.category)
}

func main() {
 // Original product
 original := &Product{name: "Phone", category: "Electronics"}
 fmt.Println(original.GetDetails()) // Output: Product Name: Phone, Category: Electronics

 // Clone the product and change its name
 cloned := original.Clone().(*Product)
 cloned.SetName("Smartphone")
 fmt.Println(cloned.GetDetails()) // Output: Product Name: Smartphone, Category: Electronics
}

```
---

## 6. Observer

Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

```go
package observer

type Observer interface {
    Update(data string)
}

type Subject struct {
    observers []Observer
}

func (s *Subject) Attach(o Observer) {
    s.observers = append(s.observers, o)
}

func (s *Subject) Notify(data string) {
    for _, observer := range s.observers {
        observer.Update(data)
    }
}
```

---

## 7. Strategy

Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

```go
package strategy

type Strategy interface {
    Execute(a, b int) int
}

type Add struct{}
func (Add) Execute(a, b int) int { return a + b }

type Subtract struct{}
func (Subtract) Execute(a, b int) int { return a - b }
```

---

## 8. Decorator

Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

```go
package decorator

type Coffee interface {
    Cost() float64
}

type SimpleCoffee struct{}

func (s *SimpleCoffee) Cost() float64 {
    return 2.0
}

type MilkDecorator struct {
    coffee Coffee
}

func (m *MilkDecorator) Cost() float64 {
    return m.coffee.Cost() + 0.5
}
```

---

## 9. Command

Encapsulate a request as an object, thereby allowing for parameterization of clients with queues, requests, and operations.

```go
package command

type Command interface {
    Execute()
}

type Light struct{}

func (l *Light) TurnOn() {
    // Logic to turn on the light
}

type TurnOnLightCommand struct {
    light *Light
}

func (c *TurnOnLightCommand) Execute() {
    c.light.TurnOn()
}
```

---

## 10. Adapter

Convert the interface of a class into another interface that clients expect. The Adapter allows classes to work together that couldn't otherwise because of incompatible interfaces.

```go
package adapter

type EuropeanPlug interface {
    Connect()
}

type USAPowerSocket struct{}

func (u *USAPowerSocket) Connect() {
    // Logic to connect
}

type EuropeanToUSAAdapter struct {
    europeanPlug EuropeanPlug
}

func (a *EuropeanToUSAAdapter) Connect() {
    a.europeanPlug.Connect()
}
```

---

## Conclusion

These design patterns can greatly enhance the architecture and maintainability of your Go applications. Choosing the right pattern depends on the specific requirements of your project and your design goals.

---
