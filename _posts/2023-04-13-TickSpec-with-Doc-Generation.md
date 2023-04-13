---
layout: post
title: "Lean BDD with Documentation Generation"
description: |
    TickSpec is a lean BDD, Gherkin based, framework with powerful F# integration. Even though Gherkin is
    designed to be human readable as well as machine readable, when it comes to providing a long term 
    specification HTML has clear benefits over Gherkin.
tags: [Testing]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

<img src="{{ site.url }}/assets/BDD/tickspec-doc-gen.png" class="dynimg" title="FSharp, TickSpec and doc generation" alt="FSharp, Gherkin, HTML"/>

Behavior driven development (BDD) and the Gherkin language, first and foremost, are about collaboration and documentation.

The key idea is to specify the behavior of a software system by describing its features using concrete scenarios
and examples. By describing the scenarios in the language of the domain and by doing this together
with the domain experts, we ensure that each feature is completely covered by its scenarios and that each 
scenario is specified correctly. 

Using this approach ensures that we build the right system.

We write the scenarios using the Gherkin language which is designed to be both, human and machine readable.
By parsing the scenarios and automating its steps we turn the specification into executable test cases 
which verify that the implemented features behave as specified.

Using this approach ensures that we build the system right.

Now, reading raw Gherkin in an IDE is fine when developing and reviewing a particular feature and its 
scenarios. But when it comes to providing a long term specification, HTML has clear benefits over Gherkin
with respect to readability of the scenarios and navigation between features.

For this reason I have added support for generating HTML documentation from Gherkin based feature files
to the [TickSpec](https://github.com/fsprojects/TickSpec) extension 
[TickSpec.Build](https://github.com/plainionist/TickSpec.Build).

<!--more-->

## Generating HTML documentation

To generate the HTML documentation run the following command:

```bash
TickSpec.Build doc ./src ./html
```

The first folder specifies the root of the source tree which should be searched for ``*.feature`` files.
The second folder specifies the location where the HTML files should be generated to.

You can safely specify the root to your complete code base, as folders containing build artifacts
like ``obj`` and ``node_modules`` are ignored.

The command will generate a separate HTML file for each feature file. The Visual Studio project local
folder structure will be preserved. Each HTML file intentionally only contains a HTML fragment of
type ``<article/>`` so that these articles can easily be integrated in an existing HTML page.

Each article is organized in different sections which provide dedicated CSS classes for styling.
Checkout the [TickSpec.Build documentation](https://github.com/plainionist/TickSpec.Build/blob/main/README.md) 
for a complete list of available CSS classes.

You can use the command line option ``--toc html`` to get a table of contents generated as a standalone HTML page.
This option will also add a reference to a CSS stylesheet to each article which can be used for defining
the CSS classes mentioned above. This stylesheet needs to be named ``style.css`` and has to be located
next to the generated table of contents file. 

Alternatively you can use the command line option ``--toc json`` to get a table of contents generated as a
JSON document which can then be used for integrating the generated HTML articles into an existing HTML
documentation or web page.

## Integration using Vue

Personally, I use this HTML generation feature in one of my projects to integrate the BDD specification 
into an existing web application based on Vue.JS.

Therefore I configure webpack to load the generated HTML articles as raw text files by adding the following
configuration to the ```vue.config.js```:

```javascript
  configureWebpack: {
    module: {
      rules: [
        {
          test: /\.html$/,
          include: [
            path.resolve(__dirname, "src/assets/spec")
          ],
          loader: 'raw-loader'
        }
      ]
    }
  }
```

Next, I created a dedicated Vue component to render the specification including the table of
contents read from the JSON document and some basic search support. 

```html
<template>
  <article>
    <sh-text>
      <h2>Table of Contents</h2>

      <input type="text" v-model="search" placeholder="Search ..."/>

      <span v-if="!articles.length">No Results Found.</span>

      <div v-else v-for="(article, index) in articles"
           :key="'article_' + index + Math.random()">
        <a @click="selected = article">{{ article.path }}</a>
      </div>

      <div v-html="selected?.html" />
    </sh-text>
  </article>
</template>

<script>
  import toc from '@/assets/spec/toc.json'
  
  export default {
    name: 'BackLookSpecs',
    data () {
      return {
        search: '',
        selected: null,
        store: []
      }
    },
    computed: {
      articles () {
        return this.store.spec
          .filter(item => {
            return item.title.toLowerCase().includes(this.search.toLowerCase()) ||
              item.html.toLowerCase().includes(this.search.toLowerCase())
          })
          .sort((a, b) => a.path < b.path ? 1 : 0)
      }
    },
    created () {
      const specFiles = require.context('@/assets/spec', true, /\.html/)
      const tocIndex = new Map(toc.map((x) => ['./' + x.folders.concat([x.filename]).join('/'), x]))

      this.store = specFiles.keys()
        .map((x) => {
          const tocEntry = tocIndex.get(x)
          return {
            ...tocEntry,
            path: tocEntry.folders.concat([tocEntry.title]).join('/'),
            html: specFiles(x).default
          }
        })
    }
  }
</script>

<style>
  .gherkin-keyword {
    color: blue;
  }

  .gherkin-tags {
    font-weight: bold;
    padding-right: 5px
  }
</style>
```

## MsBuild integration

TickSpec.Build also supports basic MsBuild integration for the HTML documentation generation feature
which is described in the [project documentation](https://github.com/plainionist/TickSpec.Build/blob/main/README.md).

In my particular project I have multiple Visual Studio projects containing BDD feature files and I have one specific
Visual Studio project which hosts the help system of the web application. In this web project I have integrated the
HTML documentation generation for all the feature files in the following way:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>

    <FeatureFileHtmlInput>..\..\</FeatureFileHtmlInput>
    <FeatureFileHtmlOutput>WebUI\src\assets\spec\</FeatureFileHtmlOutput>
    <TickSpecBuildTocFormat>json</TickSpecBuildTocFormat>
  </PropertyGroup>

  <!-- all the other content not relevant for this post -->

  <ItemGroup>
    <PackageReference Include="TickSpec.Build" Version="2.8.0" />
  </ItemGroup>

  <Target Name="ClearSpecs" BeforeTargets="GenerateFeatureFileHtml">
    <ItemGroup>
      <FilesToDelete Include="$(FeatureFileHtmlOutput)\**\*.html"/>
    </ItemGroup>
    <Delete Files="@(FilesToDelete)" />  
  </Target>

</Project>
```

## Conclusion

Using this approach I can now publish my system specification with improved readability and 
easy to use navigation features to the users of my application. Of course this specification does
not replace a real user manual but it definitively can serve as a reference in case of specific questions
on how the system behaves.

## References

- [Lean BDD and Code Generation](http://www.plainionist.net/TickSpec-with-Code-Generation/)
- [Lean BDD with even more Code Generation](http://www.plainionist.net/TickSpec-More-CodeGen/)
