---
layout: post
title: "How I made my finally block not being called (.Net)"
description: |
  It is widely expected that the compiler or the CLR takes care that the finally block of a try-finally 
  statement is called in all cases - even if an exception occurs. But this is not entirely true. 
tags: [coding-tips]
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

In .Net the source code of a ```finally``` block is always called, right?
Even if there is a ```return``` statement or an exception is thrown in the ```try``` block, correct?
I mean, this is the reason why we use ```try-finally``` in the first place.

Well, I got into a case where the code of the ```finally``` block was not called ...

<!--more-->

Let's start with an example of a ```try-finally``` statement:

```cs
public class Repository
{
    private int myTransactionsCount = 0;

    public int TransactionsCount => myTransactionsCount;

    public IEnumerable<string> FindAll()
    {
        try
        {
            myTransactionsCount++;

            yield return "a";
            yield return "b";
            yield return "c";
            yield return "d";
        }
        finally
        {
            myTransactionsCount--;
        }
    }
}
```

This code simulates the access to some database. We first increase the open transactions count, 
then return the queried data and use the ```finally``` block to ensure that the transaction counter
is decreased before returning from the method.

When we call this code we would expect that after the execution of ```FindAll``` the transaction count
is the same as before the call - assuming concurrency free execution. Here is a test case:

```cs
[Test]
public void LinQ_WillWork()
{
    var repository = new Repository();

    // just a few more calls to have higher "transaction count"
    var isNotEmpty = System.Linq.Enumerable.Any(repository.FindAll());
    isNotEmpty = System.Linq.Enumerable.Any(repository.FindAll());
    isNotEmpty = System.Linq.Enumerable.Any(repository.FindAll());

    Assert.That(isNotEmpty, Is.True);
    Assert.That(repository.TransactionsCount, Is.EqualTo(0));
}
```

This test case uses LinQ to operate on the data returned from the ```FindAll``` method and
as expected this test case goes green.

Now let's imaging we do not work directly with the enumerator returned by ```FindAll``` but
pass it on to some generic code which works with ```IEnumerable``` instead of ```IEnumerable<T>```.
In this code we use a custom implementation of LinQ's ```Any``` which supports the non-generic
```IEnumerable``` and the implementation looks like this:

```cs
public static bool Any(this IEnumerable self)
{
    return self.GetEnumerator().MoveNext();
}
```

Simple!

And here is the adapted test case which now uses our custom extension method ```Any```:

```cs
[Test]
public void CustomLinQ_WillFail()
{
    var repository = new Repository();

    // just a few more calls to have higher "transaction count"
    var isNotEmpty = AboutCleanCode.EnumerableExtensions.Any(repository.FindAll());
    isNotEmpty = AboutCleanCode.EnumerableExtensions.Any(repository.FindAll());
    isNotEmpty = AboutCleanCode.EnumerableExtensions.Any(repository.FindAll());

    Assert.That(isNotEmpty, Is.True);
    Assert.That(repository.TransactionsCount, Is.EqualTo(0));
}
```

Surprisingly this test case fails!

```
Message: 
      Expected: 0
      But was:  3
```

In order to understand the root cause we have to understand how the C# compiler 
treats a method with ```yield return``` statements. Inspecting the compiled code with
a [dnSpy](https://github.com/dnSpy/dnSpy) reveals that the generated code looks much different
than what we have typed:

```cs
public class Repository
{
  public int TransactionsCount
  {
    get { return this.myTransactionsCount; }
  }

  public IEnumerable<string> FindAll()
  {
    Repository.<FindAll>d__3 <FindAll>d__ = new Repository.<FindAll>d__3(-2);
    <FindAll>d__.<>4__this = this;
    return <FindAll>d__;
  }

  private int myTransactionsCount = 0;

  [CompilerGenerated]
  private sealed class <FindAll>d__3 : IEnumerable<string>, IEnumerable, IEnumerator<string>, IEnumerator, IDisposable
  {
    [DebuggerHidden]
    public <FindAll>d__3(int <>1__state)
    {
      this.<>1__state = <>1__state;
      this.<>l__initialThreadId = Environment.CurrentManagedThreadId;
    }

    [DebuggerHidden]
    void IDisposable.Dispose()
    {
      int num = this.<>1__state;
      if (num == -3 || num - 1 <= 3)
      {
        try
        {
        }
        finally
        {
          this.<>m__Finally1();
        }
      }
    }

    bool IEnumerator.MoveNext()
    {
      bool result;
      try
      {
        switch (this.<>1__state)
        {
        case 0:
          this.<>1__state = -1;
          this.<>1__state = -3;
          this.<>4__this.myTransactionsCount = this.<>4__this.myTransactionsCount + 1;
          this.<>2__current = "a";
          this.<>1__state = 1;
          result = true;
          break;
        case 1:
          this.<>1__state = -3;
          this.<>2__current = "b";
          this.<>1__state = 2;
          result = true;
          break;
        case 2:
          this.<>1__state = -3;
          this.<>2__current = "c";
          this.<>1__state = 3;
          result = true;
          break;
        case 3:
          this.<>1__state = -3;
          this.<>2__current = "d";
          this.<>1__state = 4;
          result = true;
          break;
        case 4:
          this.<>1__state = -3;
          this.<>m__Finally1();
          result = false;
          break;
        default:
          result = false;
          break;
        }
      }
      catch
      {
        this.System.IDisposable.Dispose();
        throw;
      }
      return result;
    }

    private void <>m__Finally1()
    {
      this.<>1__state = -1;
      this.<>4__this.myTransactionsCount = this.<>4__this.myTransactionsCount - 1;
    }

    string IEnumerator<string>.Current
    {
      [DebuggerHidden]
      get { return this.<>2__current; }
    }

    [DebuggerHidden]
    void IEnumerator.Reset()
    {
      throw new NotSupportedException();
    }

    object IEnumerator.Current
    {
      [DebuggerHidden]
      get
      {
        return this.<>2__current;
      }
    }

    [DebuggerHidden]
    IEnumerator<string> IEnumerable<string>.GetEnumerator()
    {
      Repository.<FindAll>d__3 <FindAll>d__;
      if (this.<>1__state == -2 && this.<>l__initialThreadId == Environment.CurrentManagedThreadId)
      {
        this.<>1__state = 0;
        <FindAll>d__ = this;
      }
      else
      {
        <FindAll>d__ = new Repository.<FindAll>d__3(0);
        <FindAll>d__.<>4__this = this.<>4__this;
      }
      return <FindAll>d__;
    }

    [DebuggerHidden]
    IEnumerator IEnumerable.GetEnumerator()
    {
      return this.System.Collections.Generic.IEnumerable<System.String>.GetEnumerator();
    }

    private int <>1__state;

    private string <>2__current;

    private int <>l__initialThreadId;

    public Repository <>4__this;
  }
}
```

Whenever we use ```yield return``` the compiler generates a state machine. If we search for 
the code of our ```finally``` block we realize that it was moved to a separate method


```cs
    private void <>m__Finally1()
    {
      this.<>1__state = -1;
      this.<>4__this.myTransactionsCount = this.<>4__this.myTransactionsCount - 1;
    }
```

which is only called in two cases

1. the enumerator is moved to the very end, which means the SAME enumerator is used
   to traverse the complete data (case 4)
2. ```Dispose``` is called on the enumerator after usage

As our custom ```Any``` implementation only fetches only the first item AND does NOT call Dispose
the ```<>m__Finally1``` method is not called, so our code from the finally block is NOT executed!

Looking into the implementation of LinQ's ```Any``` shows that ```Dispose``` is always called 
(note that ```IEnumerator<T>``` even derives from ```IDisposable``` so the ```using``` statement can be used):

```cs
public static bool Any<TSource>(this IEnumerable<TSource> source) {
    if (source == null) throw Error.ArgumentNull("source");
    using (IEnumerator<TSource> e = source.GetEnumerator()) {
        if (e.MoveNext()) return true;
    }
    return false;
}
```

(From: [https://github.com/microsoft/referencesource/blob/master/System.Core/System/Linq/Enumerable.cs](Microsoft Reference Source))

Now that the root cause is understood it is also clear how to fix the custom ```Any``` 
implementation:

```cs
public static bool Any(this IEnumerable self)
{
    var enumerator = self.GetEnumerator();
    try
    {
        return enumerator.MoveNext();
    }
    finally  
    {
        // important to call dispose if implemented by enumerator
        // to support "yield return" in "try-finally"
        (enumerator as IDisposable)?.Dispose();
    } 
}
```

Re-running the test case shows that the issue is fixed and the test case passes.

## What about exceptions?

If our code would throw an exception while computing the value which should be returned
using ```yield return``` then the state machine implemented by the compiler would catch
this exception and call ```Dispose``` on the enumerator as well.

```cs
    bool IEnumerator.MoveNext()
    {
      bool result;
      try
      {
        switch (this.<>1__state)
        {
          // our code would be here and might through an exception
        }
      }
      catch
      {
        this.System.IDisposable.Dispose();
        throw;
      }
      return result;
    }
```

## Sample Code

The sample code used in this post can be found at 
[Plainionist/AboutCleanCode](https://github.com/plainionist/AboutCleanCode/tree/main/EnumeratorDispose)

## Conclusion

If you call ```IEnumerable.GetEnumerator``` and work with the enumerator directly make sure 
you call ``Dispose`` when you are done!


