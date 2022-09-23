---
layout: post
title: "Automating dependency governance in .NET"
description: |
    Every software project requires some structure to manage complexity. Every structure implies rules 
    which have to be followed to not break the structure. Every rule requires automatic governance to 
    be effective. How can such automatic governance be set up in .Net projects?
tags: [design]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

Every software project of some reasonable size [needs some structure](https://youtu.be/1IE8RC-IOSE)
to manage complexity and to ensure that it [fits into every developers head](/Code-that-fits-in-your-head/).

Every structure also implies some rules on which code should go where and which dependencies are allowed,
for example

- dependencies between architectural layers are only allowed in one direction
- avoid coupling between individual components or features
- code creating "painful dependencies" should kept outside of the business logic assemblies

But how to ensure that those rules are followed in the daily business of a software project?

<!--more-->

Naive answer: The project has an architecture specification which clearly describes these rules and
the rational behind those. Every developer in the team reads this architecture specification when 
joining the team and follows those rules during feature development.

It is certainly a good starting point to have those rules clearly documented and trained to every
new developer in the team ...

BUT

... even if we assume the most disciplined team there are reasons why having good documentation
is not enough, for example:

- Mistakes happen, that's why writing down functional requirements is not enough - we write tests to 
  verify our software.
- Time pressure tends to cause compromises which result in technical debt, unless such compromises are 
  not accepted by the CI/CD pipeline.
- Developers joining the team have to learn a lot: domain, architectural rules, coding guidelines,
  processes of the project and organization and many more. 
  It is unrealistic - and maybe even unfair - to expect that all these rules, guidelines and processes 
  can be learned, memorized and followed by day one.
  Governance simply reduces load from a developers brain and memory.

## How should such governance be set up?

One theoretical approach would be a regular code review session performed by the
software architect or the complete team, analyzing whether the implementation still matches
the architecture rules.

Even though code reviews are a great tool for knowledge sharing, it is certainly not the 
best tool for regular (!) governance of rules.

Even if such review sessions are supported by tools like [NDepend](https://www.ndepend.com/), 
[SonarQube](https://www.sonarqube.org/) or [Plainion.GraphViz](http://www.plainionist.net/Plainion.GraphViz/),
this would still not be optimal as the developer would not get immediate feedback on any violation
introduced.

Clearly, such governance - maybe even every governance? - has to be automated, means it has 
to be integrated into the CI/CD pipeline to be really, practically effective.

## Automation options

### Compiler 

The first - and to some extend simplest - option to automate the dependency governance is the compiler.

The compiler will simply not compile a project if a class tries to use internal APIs of another 
class in another assembly (unless those are "friend assemblies", defined with "InternalsVisibleTo" attribute).

The compiler will also simply not compile a project if it creates cyclic dependencies to other projects.

But of course this approach has quite some limitations. We may not want to move every little component into
its own assembly just to make use of those compiler features and we can also not enforce any "semantical" rules
like layer A should not depend on layer B, this way.

### NDepend 

NDepend is probably the most used tool for analyzing .NET code. It has a very powerful query language which
probably allows implementing almost every governance rule we could think of.

NDepend can also be integrated in the CI/CD pipeline and so could be used to enforce any governance rules
during code integration.

I never tried integrating NDepend rules into Visual Studio so I do not know whether those rules could be 
integrated into the developers workflow easily so that every developer can verify any change already before
submitting. 

If you have experiences on that topic please share it in the comments.

### Custom Roslyn rules

Since Microsoft has released their compiler framework "Roslyn" it is pretty easy to implement custom
"code analyzer" which integrate seamlessly into MsBuild and so into any CI/CD pipeline.

We could develop custom code analyzers which verify the dependency rules based on source code analysis 
during the build of each project.

Here is an example on how to verify that unit test project are not referenced by any other (unit test) project:

``` csharp
[DiagnosticAnalyzer(LanguageNames.CSharp)]
public class UnitTestAssembliesMustNotBeReferenced : DiagnosticAnalyzer
{
    public const string DiagnosticId = "Architecture";

    private static DiagnosticDescriptor Descriptor { get; } =
        new DiagnosticDescriptor("AR0001", 
            "UnitTest assemblies must not be referenced by other assemblies",
            "The assembly '{0}' references the unit test assembly: {1}",
            category: "Architecture",
            defaultSeverity: DiagnosticSeverity.Error,
            isEnabledByDefault: true);

    public override ImmutableArray<DiagnosticDescriptor> SupportedDiagnostics =>
        ImmutableArray.Create(Descriptor);

    public override void Initialize(AnalysisContext context)
    {
        context.EnableConcurrentExecution();
        context.ConfigureGeneratedCodeAnalysis(GeneratedCodeAnalysisFlags.None);
        context.RegisterCompilationAction(AnalyzeCompilation);
    }

    private void AnalyzeCompilation(CompilationAnalysisContext context)
    {
        foreach (var item in context.Compilation.References)
        {
            if (Path.GetFileNameWithoutExtension(item.Display)
              .EndsWith(".UnitTests", StringComparison.OrdinalIgnoreCase))
            {
                context.ReportDiagnostic(
                  Diagnostic.Create(
                      Descriptor,
                      null,
                      context.Compilation.AssemblyName,
                      Path.GetFileNameWithoutExtension(item.Display)));
            }
        }
    }
}
```

## NsDepCop

One nice little tool I recently discovered and started using is [NsDepCop](https://github.com/realvizu/NsDepCop/).

It will be integrated in each project by simply adding a NuGet package reference and by providing a 
configuration file where allowed and disallowed dependencies can be specified based on namespaces. 

Here is a simple example, showing how to ensure that the business logic and the domain layer
remain independent from the infrastructure layer:

```xml
<NsDepCopConfig IsEnabled="true" ChildCanDependOnParentImplicitly="true">

  <Allowed From="*" To="System.*" />

  <Allowed From="App.BusinessLogic.*" To="App.Domain.*" />
  <Allowed From="App.Infrastructure.*" To="App.BusinessLogic.*" />
  <Allowed From="App.Infrastructure.*" To="App.Persistance.*" />
  
</NsDepCopConfig>
```

How exactly a governance, e.g. of the Dependency Rule in the Clean Architecture,
could be implemented using NsDepCop I will explain in the next video on my
[YouTube channel](https://www.youtube.com/c/AboutCleanCode). Stay tuned.

## Conclusion

In my reality, architectural rules - esp. dependency rules - are only effective if those come
with automatic governance. Nowadays there are various options available to automate a governance
of simple as well as highly sophisticated rules, so there is no excuse for not having any
automated governance at all ;-)

Which tools do you use to ensure that dependency rules are followed in your project?
