---
layout: post
title: "Lean BDD with even more Code Generation"
description: |
    TickSpec is a lean BDD framework with powerful F# integration but the resulting tests are not
    nicely integrated in the TestExplorer of Visual Studio. This is were code generation comes
    to the rescue.
tags: [Testing]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

<img src="{{ site.url }}/assets/BDD/tickspec-more-codegen.png" class="dynimg" title="FSharp, TickSpec and code generation" alt="FSharp, Gherkin, T4 templates"/>

Just recently, I wrote about my BDD approach in one of my projects [in this article](/TickSpec-with-Code-Generation).
I have used this setup now for a while and it actually worked quite well for me but
there is one thing which turned out to be quite annoying over time.

<!--more-->

The problem was: the Test Explorer couldn't identify the sources of a test case which means, if the test case failed,
I couldn't simply double click the sources to navigate to the test case and also starting the test case with debugger
attached was not easily possible.

<img src="{{ site.url }}/assets/BDD/no-sources.png" class="dynimg" title="Test Explorer finds no sources" alt="VS Test Explorer not finding sources for a test case"/>

The reason of this problem was that the test cases were generated at runtime using reflection and NUnits ```TestCaseSource```
mechanism which means there are simply no sources for these test cases.

```fsharp
// GENERATED DURING BUILD
namespace Specification

open TickSpec
open NUnit.Framework
open System.Reflection

[<TestFixture>]
type ``Best Effort work items explicitly accepted should be highlighted in the backlog``() = 
    inherit AbstractFeature()

    static member Scenarios = 
        AbstractFeature.GetScenarios(Assembly.GetExecutingAssembly(), 
            "AcceptingBestEffortImprovements.feature")
```

```fsharp
type AbstractFeature() =
    static member GetScenarios(assembly:Assembly, featureSource) =
        let createTestCaseData (feature:Feature) (scenario:Scenario) =
            let scenarioName =
                scenario.Parameters
                |> Seq.fold (fun acc p -> acc.Replace("<" + fst p + ">", snd p)) scenario.Name
                |> fun x -> Regex.Replace(x, "^Scenario: ", "")

            (new TestCaseData(scenario))
                .SetName(scenarioName)
                .SetProperty("Feature", feature.Name)
            |> Seq.foldBack(fun tag data -> data.SetProperty("Tag", tag)) scenario.Tags

        let definitions = new StepDefinitions(assembly.GetTypes())

        let createFeature (featureFile:string) =
            let stream = assembly.GetManifestResourceStream(featureFile)
            let feature = definitions.GenerateFeature(featureFile, stream)
            feature.Scenarios
            |> Seq.map (createTestCaseData feature)

        assembly.GetManifestResourceNames()
        |> Seq.filter(fun x -> x.EndsWith(".feature", StringComparison.OrdinalIgnoreCase))
        |> Seq.filter(fun x -> x.EndsWith("." + featureSource, StringComparison.OrdinalIgnoreCase))
        |> Seq.collect createFeature
        |> List.ofSeq

    [<TestCaseSource("Scenarios")>]
    member this.Bdd (scenario:Scenario) = 
        if scenario.Tags |> Seq.exists ((=) "ignore") then
            raise (new IgnoreException("Ignored: " + scenario.ToString()))
        try
            scenario.Action.Invoke()
        with
        | :? TargetInvocationException as ex -> 
            ExceptionDispatchInfo.Capture(ex.InnerException).Throw()
```

The only solution to this problem I saw was generating the test cases as source code at build time.

For that, I first created a parser which reads the feature title as well as the scenario titles from
the feature files of a given project.

```fsharp
let ReadFeatureFile file =
    let linesWithLineNo = 
        File.ReadAllLines(file) 
        // start counting lines with 1 as in any editor
        |> Seq.mapi(fun i l -> i + 1, l)
        |> List.ofSeq

    let grep prefix =
        linesWithLineNo 
        |> Seq.filter(fun (_,x) -> x.StartsWith(prefix, StringComparison.OrdinalIgnoreCase))
        |> Seq.map(fun (i,x) -> i, x.Trim(), x.Substring(prefix.Length).Trim())

    {
        Name = grep "Feature:" |> Seq.exactlyOne |> fun (_,_,x) -> x
        Filename = Path.GetFileName(file)
        Scenarios = 
            grep "Scenario:" 
            |> Seq.append (grep "Scenario Outline:") 
            |> Seq.map(fun (lineNo, name, title) -> 
                { 
                    Name = name
                    Title = title
                    StartsAtLine = lineNo + 1 // skip scenario title
                })
            |> List.ofSeq
    }
```

Then I had to update the [existing code generator](/TickSpec-with-Code-Generation) to also
generate test cases (methods) for each scenario. Earlier I used a 
[template engine](https://www.nuget.org/packages/Mono.TextTemplating) to generate the code but
the engine I have chosen turned out to be too limited for the new needs so I removed it again 
and now generate the code using strings and a TextWriter.

```fsharp
let writeHeader (writer:TextWriter) =
    writer.WriteLine("namespace Specification");
    writer.WriteLine()
    writer.WriteLine("open System.Reflection")
    writer.WriteLine("open NUnit.Framework")
    writer.WriteLine("open TickSpec.CodeGen")
    writer.WriteLine()

let writeTestCase (writer:TextWriter) featureFile scenario =
    writer.WriteLine($"    [<Test>]")
    writer.WriteLine($"    member this.``{scenario.Title}``() =")
    writer.WriteLine($"#line {scenario.StartsAtLine} \"{featureFile}\"")
    writer.WriteLine($"        this.RunScenario(scenarios, \"{scenario.Name}\")")
    writer.WriteLine()

let writeTestFixture (writer:TextWriter) feature =
    writer.WriteLine($"[<TestFixture>]")
    writer.WriteLine($"type ``{feature.Name}``() = ")
    writer.WriteLine($"    inherit AbstractFeature()")
    writer.WriteLine()
    writer.WriteLine($"    let scenarios = AbstractFeature.GetScenarios(")
    writer.WriteLine($"        Assembly.GetExecutingAssembly(), \"{feature.Filename}\")")
    writer.WriteLine()

    feature.Scenarios
    |> Seq.iter (writeTestCase writer feature.Filename)
```

For now this is a feasible approach as most of the actual logic, needed to 
find and execute a particular scenario using [TickSpec](https://github.com/fsprojects/TickSpec),
is still in a base class similar to the one shown above.

Finally, I even added ```#line``` directives pointing to the feature file instead of the
generated "code behind" which causes VS Test Explorer to navigate the particular scenario in the
feature file when double clicking the test case.

```fsharp
[<TestFixture>]
type ``Highlight accepted BestEffort work items``() = 
    inherit AbstractFeature()

    let scenarios = AbstractFeature.GetScenarios(
        Assembly.GetExecutingAssembly(), "AcceptingBestEffortImprovements.feature")

    [<Test>]
    member this.``Rendering the initiative backlog``() =
#line 4 "AcceptingBestEffortImprovements.feature"
        this.RunScenario(scenarios, "Scenario: Rendering the initiative backlog")

    [<Test>]
    member this.``Rendering the team backlog``() =
#line 11 "AcceptingBestEffortImprovements.feature"
        this.RunScenario(scenarios, "Scenario: Rendering the team backlog")

    [<Test>]
    member this.``Rendering the team improvements backlog``() =
#line 18 "AcceptingBestEffortImprovements.feature"
        this.RunScenario(scenarios, "Scenario: Rendering the team improvements backlog")
```

The "upgraded" approach is available as individual [GitHub project](https://github.com/plainionist/TickSpec.Build)
as well as [NuGet package](https://www.nuget.org/packages/TickSpec.Build/).

Give it a try in your next F# and BDD project and let me know how it works for you.
