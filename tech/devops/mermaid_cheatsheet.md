# 🎨 Mermaid Diagrams Cheat Sheet

Mermaid is a **text-based diagramming tool** that allows users to create **flowcharts, sequence diagrams, Gantt charts, and more** using simple syntax.

📌 **Official Documentation**: [Mermaid Docs](https://mermaid-js.github.io/)  
📌 **Live Editor**: [Mermaid Live Editor](https://mermaid-js.github.io/mermaid-live-editor/)  

---

## **1. Basic Flowchart**  

### **Diagram Output:**
```mermaid
graph TD;
    A[Start] --> B[Process];
    B --> C{Decision?};
    C -- Yes --> D[Action 1];
    C -- No --> E[Action 2];
    D --> F[End];
    E --> F;
```

### **Code Example:**
````markdown
```mermaid
graph TD;
    A[Start] --> B[Process];
    B --> C{Decision?};
    C -- Yes --> D[Action 1];
    C -- No --> E[Action 2];
    D --> F[End];
    E --> F;
```
````

🔗 **More on Flowcharts**: [Flowchart Docs](https://mermaid-js.github.io/mermaid/#/flowchart)  

---

## **2. Sequence Diagram**  

### **Diagram Output:**
```mermaid
sequenceDiagram;
    participant A as Alice
    participant B as Bob

    A->>B: Hello Bob, how are you?
    B->>A: I'm good Alice, thanks!
```

### **Code Example:**
````markdown
```mermaid
sequenceDiagram;
    participant A as Alice
    participant B as Bob

    A->>B: Hello Bob, how are you?
    B->>A: I'm good Alice, thanks!
```
````

🔗 **More on Sequence Diagrams**: [Sequence Docs](https://mermaid-js.github.io/mermaid/#/sequenceDiagram)  

---

## **3. Gantt Chart**  

### **Diagram Output:**
```mermaid
gantt;
    title Project Timeline
    section Development
    Task 1 :a1, 2024-03-01, 5d
    Task 2 :after a1, 3d
    section Testing
    Unit Tests :2024-03-10, 4d
    Integration Tests :2024-03-14, 3d
```

### **Code Example:**
````markdown
```mermaid
gantt;
    title Project Timeline
    section Development
    Task 1 :a1, 2024-03-01, 5d
    Task 2 :after a1, 3d
    section Testing
    Unit Tests :2024-03-10, 4d
    Integration Tests :2024-03-14, 3d
```
````

🔗 **More on Gantt Charts**: [Gantt Docs](https://mermaid-js.github.io/mermaid/#/gantt)  

---

## **4. Pie Chart**  

### **Diagram Output:**
```mermaid
pie
    title Sales Data
    "Product A": 40
    "Product B": 30
    "Product C": 20
    "Other": 10
```

### **Code Example:**
````markdown
```mermaid
pie
    title Sales Data
    "Product A": 40
    "Product B": 30
    "Product C": 20
    "Other": 10
```
````

🔗 **More on Pie Charts**: [Pie Chart Docs](https://mermaid-js.github.io/mermaid/#/pie)  

---

## **5. State Diagram**  

### **Diagram Output:**
```mermaid
stateDiagram-v2
    [*] --> Init
    Init --> Running
    Running --> Stopped
    Stopped --> [*]
```

### **Code Example:**
````markdown
```mermaid
stateDiagram-v2
    [*] --> Init
    Init --> Running
    Running --> Stopped
    Stopped --> [*]
```
````

🔗 **More on State Diagrams**: [State Diagram Docs](https://mermaid-js.github.io/mermaid/#/stateDiagram)  

---

## **6. Class Diagram**  

### **Diagram Output:**
```mermaid
classDiagram
    class Car {
        +String model
        +int year
        +startEngine()
    }
    class ElectricCar {
        +int batteryCapacity
        +chargeBattery()
    }
    Car <|-- ElectricCar
```

### **Code Example:**
````markdown
```mermaid
classDiagram
    class Car {
        +String model
        +int year
        +startEngine()
    }
    class ElectricCar {
        +int batteryCapacity
        +chargeBattery()
    }
    Car <|-- ElectricCar
```
````

🔗 **More on Class Diagrams**: [Class Diagram Docs](https://mermaid-js.github.io/mermaid/#/classDiagram)  

---

### **Final Thoughts**  
Mermaid.js is **a powerful tool for visualizing processes and relationships** in **flowcharts, sequence diagrams, Gantt charts, and more**. It is **easy to integrate into documentation, Markdown, and web applications**.

### **Happy Diagramming with Mermaid! 🎨🚀**  
