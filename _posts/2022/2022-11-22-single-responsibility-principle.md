---
title: Single-Responsibility Principle (SRP)
image: /assets/img/default-banner.jpg
author: jack
date: 2022-11-22 20:55:00 +0800
categories: [blog, software engineering]
tags: [software engineering]
pin: false
---

If a class has too many responsibilities (High Cohesion), all classes that depend on this class have to be modified when we want to add a new feature to the class, which increases the difficulty of maintenance.

Here is an example, first define the data structure of an order, which can be output by a photocopier.

```csharp
public class Order
{
    private string id;
    private double price;

    public void Print()
    {
        ...
    }
}
```

Analyze class `Order`, it should only focus on processing the information of an order, `Print()` seems to be beyond its responsibility. Suppose there is a new requirement: export as PDF, then `Order` will be modified as:

```csharp
public class Order
{
    private string id;
    private double price;

    public void Print()
    {
        ...
    }

    public void PrintByPdfExporter()
    {
        ...
    }
}
```

At this point, all code that call to `Print()` must then be rewritten to be compatible with `PrintByPdfExporter()`. That's not a good practice when we calls `Print()` a lot in our code.

Now we apply the SRP principle to `Order`. Define an interface called `IPrinter`, then let `PdfExporter` and `Printer` implement `IPrinter`:

```csharp
public interface IPrinter
{
    void Print(string id, double price);
}

public class PdfExporter : IPrinter
{
    public void Print(string id, double price)
    {
        ...
    }
}

public class Printer : IPrinter
{
    public void Print(string id, double price)
    {
        ...
    }
}

public class Order
{
    private string id;
    private double price;
    private IPrinter printer;

    public Order()
    {
        this.printer = new PdfExporter();
        // this.printer = new Printer();
    }

    public Print()
    {
        this.printer.Print(this.Id, this.Price);
    }
}
```

This way, all changes related to printing orders are confined to individual IPrinter implementations. We only need to modify the contructor of `Order` or use dependency injection to decide what printer we what to use.
