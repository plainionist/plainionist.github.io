---
layout: post
title: "Making The Invalid Unrepresentable - Part 2"
description: |
  Following the principles of Domain Driven Design (DDD) we want to model all business rules
  explicitly in the design. As most object oriented programming languages do not support
  OR-types directly we have to be a bit creative ...
tags: [DomainDrivenDesign]
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

After having created this video

<iframe width="560" height="315" src="https://www.youtube.com/embed/IoUsyEyWW0k" 
  title="Making The Invalid Unrepresentable | Domain Driven Design" frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen>
</iframe>

on how to "make invalid cases unrepresentable" when implementing the domain model in 
object oriented programming languages, I realized we could actually take it one step further.

<!--more-->

The design we concluded with in the video was based on inheritance combined 
with higher order functions. It encapsulates the handling of the individual cases
to avoid code duplication and to allow evolvability in a safe way
(more on this in the video).

This is what we came up with

```cs
class CellValue
{
    public void Match(Action<Id> onId, Action<WildcardId> onWildcardId, 
      Action<EmptyCell> onEmptyCell)
    {
        if (this is Id id) onId(id);
        else if (this is WildcardId wildcard) onWildcardId(wildcard);
        else if (this is EmptyCell empty) onEmptyCell(empty);
        else throw new NotSupportedException($"Unknown CellValue: {this.GetType()}");
    }

    public T Select<T>(Func<Id, T> onId, Func<WildcardId, T> onWildcardId, 
        Func<EmptyCell, T> onEmptyCell)
    {
        if (this is Id id) return onId(id);
        else if (this is WildcardId wildcard) return onWildcardId(wildcard);
        else if (this is EmptyCell empty) return onEmptyCell(empty);
        else throw new NotSupportedException($"Unknown CellValue: {this.GetType()}");
    }
}

class Id : CellValue
{
    public int Number { get; init; }
    public string Kind { get; init; }
    public int Version { get; init; }
}

class WildcardId : CellValue
{
    public int Number { get; init; }
    public string Kind { get; init; }
}

class EmptyCell : CellValue { }
```

and it would be used like this

```cs
private void Print(CellValue value)
{
    value.Match(
        id => Console.WriteLine($"{id.Number}-{id.Kind}-{id.Version}"),
        wildcard => Console.WriteLine($"{wildcard.Number}-{wildcard.Kind}-*"),
        empty => Console.WriteLine($"<empty>")
    );
}
```

The evolvability of this design relies on the idea that in the entire code 
base only the higher order functions are used to work with derived types of
the ```CellValue```. As adding a new case would require an adaption of the 
higher oder functions this would result in a compile breaking change which
ensures that finally all relevant source code would have been adapted. 

Obviously this design can neither enforce the usage of the higher order functions
nor can it forbid the creation of new derived classes elsewhere in the code base.

Here is one idea how could address these flaws:

```cs
class CellValue
{
    public static CellValue Id(int number, string kind, int version) => 
        new IdCell { Value = new Id { Number = number, Kind = kind, Version = version } };
    public static CellValue WildcardId(int number, string kind) => 
        new WildcardIdCell { Value = new WildcardId { Number = number, Kind = kind } };
    public static CellValue Empty() => new EmptyCell();

    private CellValue() {}

    private class IdCell : CellValue { public Id Value; }
    private class WildcardIdCell : CellValue { public WildcardId Value; }
    private class EmptyCell : CellValue { }

    public void Match(Action<Id> onId, Action<WildcardId> onWildcardId, Action onEmptyCell)
    {
        if (this is Id id) onId(id);
        else if (this is WildcardId wildcard) onWildcardId(wildcard);
        else if (this is EmptyCell empty) onEmptyCell();
        else throw new NotSupportedException($"Unknown CellValue: {this.GetType()}");
    }

    public T Select<T>(Func<Id, T> onId, Func<WildcardId, T> onWildcardId, Func<T> onEmptyCell)
    {
        if (this is Id id) return onId(id);
        else if (this is WildcardId wildcard) return onWildcardId(wildcard);
        else if (this is EmptyCell empty) return onEmptyCell();
        else throw new NotSupportedException($"Unknown CellValue: {this.GetType()}");
    }
}

class Id
{
    public int Number { get; init; }
    public string Kind { get; init; }
    public int Version { get; init; }
}

class WildcardId
{
    public int Number { get; init; }
    public string Kind { get; init; }
}
```

As the constructor of ```CellValue``` is now private, no new cases can be added elsewhere
in the code base without adapting the actual design.

The new design has two additional nice side effects:

- New cases are created with factory methods on the ```CellValue``` which improves habitability 
  as intellisense shows all possible cases 

- The classes representing the actual values of the different cases no longer need to derive
  from ```CellValue``` which makes the design of these classes more flexible 


What do you think about this design?

Pls share in the comments below ...
