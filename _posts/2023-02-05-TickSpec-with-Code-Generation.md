---
layout: post
title: "Lean BDD and Code Generation"
description: |
    TickSpec is a lean BDD framework with powerful F# integration but the resulting tests are not
    nicely visualized in the TestExplorer of Visual Studio. This is were code generation comes
    to the rescue.
tags: [Testing]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

<img src="{{ site.url }}/assets/BDD/tickspec-codegen-header.png" class="dynimg" title="FSharp, TickSpec and code generation" alt="FSharp, Gherkin, T4 templates"/>

As already mentioned in [other posts](/Implementing-Clean-Architecture), one of my projects is a web application
which aims to bring maximum transparency into backlogs of agile teams. Over time this application grew quite a bit,
accumulated quite some features and got used by more than two dozen teams.

Of course, one important strategy to ensure the quality of this application is test automation. I never followed the classic
testing pyramid which is based on tons of classic unit tests but rather focused on describing features and scenarios in
a kind of a [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) style.

Favoring pragmatic setups, I wrote those tests without any "real" BDD framework which worked quite well for quite
some time, but in the recent weeks and months I realized that my setup needs some improvement.

So I decided to invest some time, do some evaluation and start migrating my tests to a "real" BDD framework
using feature files written in [Gherkin language](https://en.wikipedia.org/wiki/Cucumber_(software)#Gherkin_language).

And this is how my journey went so far ...

<!--more-->

# From Internal DSL to Gherkin

As already said, so far I tried practicing BDD without any framework based on feature files and Gherkin language.
Even though the feature files look nice and are easy to read, even by non-technical stakeholders, tests written 
using this approach have a severe drawback: there is no compiler which ensures that the steps, used to describe a 
scenario are actually compatible. This means, we can only detect at run-time that e.g. the "output" produced
by a particular GIVEN step is not compatible to the "input" required by a particular WHEN step.

For me, valuing a lot the idea of avoiding bugs by design and with the help of the compiler (type safety) 
where ever possible, this is definitively a severe drawback.

The alternative to an external Domain Specific Language (DSL) like Gherkin is an internal DSL which is based on 
the capabilities of a general purpose programming language. Hence, such a DSL can benefit from the features 
of the underlying programming language, like the compiler, but at the same time is also limited to syntax of
this language.

As the application is developed in F# and F# provides some really nice features which make it very convenient to 
create internal DSLs (e.g. identifiers allowing spaces, operator overloading), to me it felt like an obvious 
consequence to describe all features and scenarios in F# directly.

Here is an example of such a scenario:

```fsharp
[<TestFixture>]
module ``Reading remaining work`` =

    [<Test>]
    let ``WorkItem not implemented/done, empty Remaining Work treated as missing estimation`` () =
        WorkItem.Create(WorkItemTypeNames.WorkPackage, [
            WorkItems.Fields.State, "In Work"
            WorkItems.Fields.RemainingWork, null
        ])
        |> Read.RemainingWork
        |> should equal None

        WorkItem.Create(WorkItemTypeNames.UserStory, [
            WorkItems.Fields.State, "In Work"
            WorkItems.Fields.Tags, "EST CL MEDIUM"
        ])
        |> Read.RemainingWork
        |> should equal None
```

Even though I used this approach successfully for years to ensure the quality of the 
application, I never managed to reach a readability of these scenarios such that
these could have served as "requirement documentation" as well. This may seem 
kind of obvious to you but when I started using an F# based DSL, I was optimistic to 
develop a syntax which could be understood by other stakeholders as well, 
without any prior F# knowledge. Well, maybe I was a bit too optimistic ;-)

Time to evaluate the alternative ...

# Specflow vs TickSpec

The almost standard framework and tool for Gherkin based BDD in .NET is [SpecFlow](https://specflow.org/).
It is definitively a great BDD framework and I even use it in another, C# based, project.
Unfortunately the F# support is quite limited, so I continued my research and finally found 
[TickSpec](https://github.com/fsprojects/TickSpec) which even claims in the project description
on GitHub to provide a "powerful F# integration".

The "killer feature" from my perspective: TickSpec allows passing values "directly" from one step 
to the next. There is still no compiler which ensures that the steps used are actually compatible but
at least the required "inputs" of a step are make explicit on its "API surface".

# Getting Started with TickSpec

Getting started with TickSpec turned out to be pretty simple. I installed the respective
[NuGet package](https://www.nuget.org/packages/TickSpec) and wrote my first Gherkin based scenario:

```gherkin
Feature: Best Effort work items explicitly accepted should be highlighted in the backlog

Scenario: Rendering the initiative backlog
	GIVEN an improvement
	AND BacklogSet is set to 'Best Effort'
	AND Decision is set to 'accepted'
	WHEN rendering the initiative backlog
	THEN the BacklogSet column is highlighted
```

The respective feature file I included in the Visual Studio project ".fsproj" as embedded resource:

```xml
    <EmbeddedResource Include="AcceptingBestEffortImprovements.feature" />
```

To automate this scenario I provided so called "step definitions" in a simple F# module (comparable 
to a static class in C#). The module name doesn't have to follow a specific convention and steps can be 
organized in multiple modules.

Here are some of these step definitions:

```fsharp
// returns a IWorkItem
let [<Given>] ``an improvement`` () =
    WorkItem.Create(WorkItemTypeNames.Improvement, [
        WorkItems.Fields.IterationPath, VB99A.PlanningSheet.ReleaseIterationPath
    ])

// requires a IWorkItem e.g. from previous step
let [<Given>] ``BacklogSet is set to '(.*)'`` (value:string) (wi:IWorkItem) =
    wi.With(WorkItems.Fields.BacklogSet, value)
```

To execute such a scenario using one of the popular (unit) test frameworks like NUnit or XUnit
the TickSpec project suggests to create a generic test fixture which loads all scenario of all
feature files at run-time and dynamically builds test cases using e.g. 
[NUnit's TestCaseSource feature](https://docs.nunit.org/articles/nunit/writing-tests/attributes/testcasesource.html).

I copied the provided 
[FeatureFixture.fs for NUnit](https://github.com/fsprojects/TickSpec/blob/master/Examples/ByFramework/NUnit/FSharp.NUnit/FeatureFixture.fs)
and got my first scenarios running within a few minutes.

# Executing the Specification

I executed my scenarios in Visual Studio using the built-in Test Explorer. The scenarios turned
green and it confirmed that passing data from one step to the other work as promised.

But this approach had one ugly flaw:

<img src="{{ site.url }}/assets/BDD/TestExplorer-before.png" class="dynimg" title="Test organization in Test Explorer" alt="VS Test Explorer showing organization of the generated test cases"/>

As there was just a single test fixture which dynamically creates test cases, all scenarios of all 
features got grouped together and visualized in the Test Explorer below a single test class.
As a workaround I could have configured the Test Explorer to group the test cases (scenarios) by "traits"
(the generic test fixture provides the feature description as a property to NUnit which is interpreted by the Test Explorer as a "trait").
But as I still had lots of tests following my previous approach this workaround didn't felt convenient.

I did some experiments with the [TestCaseSource](https://docs.nunit.org/articles/nunit/writing-tests/attributes/testcasesource.html)
and the [TestFixtureSource](https://docs.nunit.org/articles/nunit/writing-tests/attributes/testfixturesource.html) feature of NUnit
but I didn't found a way get the scenarios visualized as parts of the respective feature.

The only feasible approach seamed to convert the generic test fixture into a base class and to create a derived
class for each feature file which would then pass the name of the feature file explicitly from which the scenarios 
should be read and test cases should be created. I could then copy the feature description from each feature file 
and use it as name of the respective derived test fixture.

Simple, but a clear violation of the DRY principle. 

And this is where code generation comes into the picture ...

# Code Generation

I recently read about [Source Generators](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview)
in .NET 6 but unfortunately these only support C#.

From some other projects long, long ago I remembered "T4 templates" and some more research
revealed [Mono.TextTemplating](https://github.com/mono/t4) which "started out as an open-source reimplementation
of the Visual Studio T4 text templating engine, but has since evolved to have many improvements over the original,
including support for C# 10 and .NET 6." (from the projects README.md).

It felt like a perfect fit so I decided to give it a try.

As mentioned above, I converted the generic test fixture into a base class which takes the name of the relevant 
feature file as parameters. This means the T4 template simply would have to 

- find all the "*.feature" files in the project
- create a derived class for each feature file
- extract the name of each derived class from the respective feature file 

After a few attempts this is the template I came up with:

```t4
<#@ template language="C#" hostspecific="true" #>
<#@ import namespace="System.IO" #>
<#@ import namespace="System.Linq" #>
<#@ parameter name='featuresFolder' #>

namespace Specification

open TickSpec
open NUnit.Framework
open System.Reflection

<# foreach(var featureFile in Directory.GetFiles(featuresFolder, "*.feature")) { #>
<#      
    var fileName = Path.GetFileName(featureFile); 
    var title = File.ReadAllLines(featureFile)
        .Select(x => x.Trim())
        .First(x => x.StartsWith("Feature: ", StringComparison.OrdinalIgnoreCase)); 
    title = title.Substring("Feature: ".Length);
#>
[<TestFixture>]
type ``<#= title #>``() = 
    inherit AbstractFeatures()

    static member Scenarios = AbstractFeatures.GetScenarios(Assembly.GetExecutingAssembly(), "<#= fileName #>")
<# } #>
```

Note that I had to specify ``hostspecific="true"```` in order to be able to pass parameters to the template.

[Mono.TextTemplating](https://github.com/mono/t4) provides a command line tool "dotnet-t4" to generate code 
using a T4 template but to get the code generation easier integrated into the build process of my project and 
to have more flexibility later on, I decided to use the library and host the template engine in my own CLI.

First, I tried [Mono.TextTemplating nuget](https://www.nuget.org/packages/Mono.TextTemplating) package in a 
F# based command line program but this failed at run-time, complaining that the proper ".NET hosting" was not
found. A quick research didn't reveal any solution. The 
[project documentation](https://github.com/mono/t4/blob/main/Mono.TextTemplating/readme.md) also offered a
[Mono.TextTemplating.Roslyn](https://www.nuget.org/packages/Mono.TextTemplating.Roslyn) package which 
"can be used to bundle a copy of the Roslyn C# compiler and host it in-process. This may improve template
compilation performance when compiling multiple templates, and guarantees a specific version of the compiler."
So I converted my program to C# and installed this package instead. 
The core logic are just these few lines of code:

```csharp
var template = args[0];
var output = args[1];

Console.WriteLine($"Generating '{template}' -> '{output}'");

var generator = new TemplateGenerator();
generator.AddParameter(null, null, "featuresFolder", Path.GetDirectoryName(output));
var success = generator.ProcessTemplateAsync(template, output).Result;

if (!success)
{
    foreach(var error in generator.Errors)
    {
        Console.Error.WriteLine(error);
    }
}

return success ? 0 : 1;
```

I successfully executed the program on the command line, passing the template and the output file name as 
arguments. The generated derived test fixture looked like this:

```fsharp
namespace Specification

open TickSpec
open NUnit.Framework
open System.Reflection

[<TestFixture>]
type ``Best Effort work items explicitly accepted should be highlighted in the backlog``() = 
    inherit AbstractFeatures()

    static member Scenarios = 
        AbstractFeatures.GetScenarios(Assembly.GetExecutingAssembly(), 
            "AcceptingBestEffortImprovements.feature")
```

When I executed my scenarios in the Test Explorer again I got those grouped and visualized as expected: 
scenarios below the respective feature and the features (test classes) properly named.

<img src="{{ site.url }}/assets/BDD/TestExplorer-after.png" class="dynimg" title="Test organization in Test Explorer" alt="VS Test Explorer showing organization of the generated test cases"/>

# Build Integration

The final step left was to integrate my little code generator into the build process of my project so that 
new derived classes get generated whenever I create a new feature file. 
The simples approach was to create a new target in the test project and use the ```BeforeTargets``` attribute to hook
it into the right place of the build process:

```xml
<Target Name="GenerateFeatures" BeforeTargets="BeforeBuild;BeforeRebuild">
  <Exec Command="$(MSBuildProjectDirectory)\..\Build.CodeGen\bin\Debug\Build.CodeGen.exe $(MSBuildProjectDirectory)\FeatureFixture.tt" Outputs="FeatureFixture.fs">
    <Output ItemName="Generated" TaskParameter="Outputs" />
  </Exec>
  <ItemGroup>
    <FileWrites Include="@(Generated)" />
  </ItemGroup>
</Target>
```

Hint: "FileWrites" tells MsBuild that these files should be cleaned up during a "clean build".

Of course this build script can be further improved, e.g. to support incremental compile and 
certainly should be factored out so that it can be reused for other test projects in the code base.
These improvements and further details I have covered in this video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/s-4ogN6B5x8" 
    title="YouTube video player" frameborder="0" 
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen>
</iframe>

# Conclusion

And this is the current status of my BDD, Gherkin & TickSpec journey.

Small add-on: As Visual Studio (which I mostly use for this project) doesn't support proper
syntax highlighting for T4 and Gherkin I installed the
[OpeOpen in Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=MadsKristensen.OpeninVisualStudioCode)
extension and edit these files in VS Code using the
[T4 Support](https://marketplace.visualstudio.com/items?itemName=zbecknell.t4-support)
and
[Cucumber (Gherkin) Full Support](https://marketplace.visualstudio.com/items?itemName=alexkrechik.cucumberautocomplete)
extensions.

And with this setup I now feel fully enabled to write all new tests using the new Gherkin and
TickSpec based approach and I will also migrate all existing scenarios to the new approach step-by-step.

**Update:** The journey continues [here](/TickSpec-More-CodeGen)
