
# Beyond Data Mesh

Data Mesh is a major advance over past architectures. It's the beginning, not the end of where we need to go. What do we need to do beyond the current Data Mesh architecture? 
In the picture below, the left side is copied from: [Data Mesh Principles and Logical Architecture](https://martinfowler.com/articles/data-mesh-principles.html). It shows some nodes in a data mesh with APIs for access. The left side highlights the idea of the domain boundary an
As highlighted in the picture below[^original-data-mesh-material], Data Mesh is about 1) finding the right boundaries around apps and data and 2) putting stable access in place at those boundaries via different types of APIs. This article discusses the things we can do to find those boundaries and going beyond that to really enable the business. 

![data mesh boundaries](./images/data-mesh-boundaries.png)

  [^original-data-mesh-material]: 

What I've seen of data mesh approaches is to focus on the data products that make up the boundary of the nodes in the mesh. This is starting too much at the end deliverable. The place to start is the boundaries. The secret to finding the boundaries is the ubiquitous language. The Domain Driven Design (DDD) community proposes various [ways to discover, document, and visualize the ubiquitous language](https://www.linkedin.com/advice/0/how-do-you-document-communicate-your-ubiquitous):

>The ubiquitous language is not just a set of terms or jargon, but a shared understanding of the domain and its problems. It reflects the domain model, which is the conceptual representation of the domain in code. The ubiquitous language and the domain model should evolve together, as the developers and the business experts learn more about the domain and refine their solutions. The ubiquitous language is important because it enables collaboration, alignment, and clarity among the different roles and perspectives involved in the software project.

We use the language to find the boundaries. The boundaries are at the points where the language of the domain’s data models and processing changes. This is where we want to put things like data products and APIs. Using the language also enables us to clearly understand what's going on behind those boundaries. 

## Domain Languages

This level of combining data product and DDD thinking is a good start. Really powerful capabilities are found by going beyond this. The DDD approach stops at things like, dictionaries, context maps, and living documentation is a dynamic form of documentation generated from source code and tests. We need to go beyond that and formalize the language.  That may sound cryptic or scary but it's not because there is a well established discipline and community for building Domain Specific Languages (DSLs) to help us and we don't need to go all the way to implementing a DSL to get a lot of the power we're after.[^DSL-community] 

[^DSL-community]: Great places to start with DSLs are this community: [Subject Matter First](https://subjectmatterfirst.org/) and the writtings of this master practitioner: [the further reading list after this article](https://www.linkedin.com/pulse/relationship-between-domain-driven-design-languages-markus-voelter/) or just google for anything written by Markus Voelter.

To start to get benefits from defining a language requires at least a minimal formalization. Formalizing the language means we work with the business users that operate in the domain to create a structured version of their ubiquitous language. This takes the form of a basic language structure and syntax.  The syntax is built on existing notations and conventions used in the domain, e.g., text, tables, symbols, and diagrams, not just lots of keywords and curly braces. It requires becoming very clear – formal! – about the concepts that go into the language. In fact, building the language, because of the need for formalization, helps you become clear about the concepts of the domain in the first place. The benefits of the approach happen right away because language definition acts as a catalyst for understanding the domain![^Markus-adapted]

[^Markus-adapted]: This content is adapted from articles written by [Markus Voelter](https://voelter.de/index.html). 

The previous paragraph is either sounding exciting or scary, depending on where you are on the idea of programming languages and the tech for building languages in general and DSLs in particular. Defining and potentially implementing a true DSL is the way to get to the ultimate in power and differentiation of a data mesh solution. Lets look at options for a first step short-cut to our desired result.

[Dbt](https://www.getdbt.com/) is one of the best technologies to get to a data mesh and move along the path to a DSL for the ubiquitous language approach proposed in this article. I'm not affiliated with the company behind dbt. I'm using it as a concrete example of the features needed to take this approach. That said, I am advocating for it's use because I really like it. 

For those not familiar with dbt, the important things about it for this discussion are:
- *Models* - Each model lives in a single file and contains logic that either transforms raw data into a dataset that is ready for analytics or, more often, is an intermediate step in such a transformation.
- *Sources* - A way to name and describe the data loaded into your warehouse by your Extract and Load tools.
- *Tests* - built from SQL queries that you can write to test the models.
- *Exposures* - A way to define and describe a downstream use of your project.
- *Macros* - Blocks of code written in Jinja, a templating language that you can reuse multiple times.

The following pictures show examples of some dbt configuration language files. The first is a dbt model which is nothing more than SQL.

<img src="./images/dbt-model.png" alt="dbt model config" width="50%">

The next picture shows the template for the additional configuration of a model.

<img src="./images/dbt-model-properties.png" alt="dbt model config" width="30%">

Models are built by accessing the data exposed by other models or source. This is the core of a data API for a data mesh domain. You formally create your data products by *exposing* them. Dbt has some basic features to control access, e.g., Exposures, and they advancing those features rapidly. 

You could just use the out-of-the-box dbt features and implement a reasonable data mesh. All of it is written in text files and taken together it forms a domain language for your data transformation and access workflows. It's a low level and business domain independent language rather than a domain specific language. 

## Dbt Macros as the Beginning of a DSL

We move to being the move to a DSL through the use of dbt macros. Macros, written using dbt's Jinja features, are pieces of code that can be reused multiple times. Using macros we can build higher-level abstractions that are specific to the business domain. For example, rather than having analysts creating new data products rewrite the complex logic for combining data for some specific operation in the domain we can write it once as a macro and simplify and standardize that part of the operation. Programmers look at this as simply not repeating ourselves (DRY). It's more important to design the macros so they align with the ubiquitous language of the domain than to do it for technical reasons. There are significant limits to what we can do with macros and there is still a lot of dbt complexity and detail exposed. However, for the right audience domain specific macros can still be a major step forward. 

Using this approach the data mesh architecture looks like the following.

![dbt in mesh node](./images/dbt-in-mesh-node.png)

The exposed dbt models are serving the **D**ata API. That access can be via raw SQL or by creating new dbt models outside the domain boundary that use a data product. At the end of this article, I'll revisit the question of whether the Apps are inside the domain boundary. In the data mesh implementations I've seen, it's mostly been the apps as external data sources to a cloud-based data mesh. I feel we need to take a more whole-enterprise view of the bounded domains that makeup the data mesh so the apps should sometimes be inside. 

## Semantic Layer 

The next step up the DSL ladder is already part of dbt: the [dbt Semantic Layer](https://www.getdbt.com/blog/dbt-semantic-layer-whats-next/). "The dbt Semantic Layer allows data teams to centrally define essential business metrics like revenue, customer, and churn in the modeling layer (your dbt project) for consistent self-service within downstream data tools like BI and metadata management solutions. The dbt Semantic Layer provides the flexibility to define metrics on top of your existing models and then query those metrics and models in your analysis tools of choice."[^dbt-semantic-layer]. This layer is a language for defining metrics. dbt talks about its value from the technical perspective. We're looking it as another part of our domain specific language. The business surely includes a lot in their ubiquitous language about the metrics, e.g., how are they named, how are they calculated, how do they evolve over time and where are they used. The following shows an example of a metric defined in the dbt language.

![example metric](images/dbt-metric-example.png)

The following shows how the semantic layer fits into business use.

![dbt semantic layer architecture](images/dbt-sl-architecture.png)

Examples of the kinds of metrics that can be expressed in the language:
- Expressions: (eg. transactions – cancellations)
- Ratios: (eg. revenue per customer)
- Cumulative Metrics (eg. weekly active users)
- Aggregation types (such as sum_boolean and percentile)

I see the value of a central definition of metrics in a semantic layer as transformative for a business. It will have dramatic effects on standardizing everything from basic BI reporting to the most advanced AI. The fact that the business can now see the definition 

As for all the rest of dbt, the metrics language is low level and generic rather than a business DSL. Once the metrics are defined, using them in combination with the domain specific dbt macros is a significant step forward.

  [^dbt-semantic-layer]: See: https://docs.getdbt.com/docs/use-dbt-semantic-layer/dbt-semantic-layer

## Full DSL

There are limits to how well we can model the ubiquitous language of the business using dbt or similar generic tools. Our ability to really model the language becomes possible when we formalize our understanding of the language of the domain as a DSL. With the infrastructure of something like dbt we can have the DSL generate dbt configurations that do what the semantics of the DSL specify. The DSL isn't limited to just generating dbt. It would generate whatever is needed to perform the DSL statements. The following figure introduces the what this would look like.

![full DSL architecture](./images/intro-to-dsl-architecture.png)

The following are concrete examples of formalization of the language of the domain from my recent projects, see [Real Examples](#this-isnt-just-theory) for more extensive discussion of real projects using DSLs. The first example is  go from simple and more technical language structures to more complex and more domain specific:

- Data Vault - The use of a data vault[^data-vault] is a great data structure for use inside domains of the data mesh. Covering data vaults requires a separate article. For this example, just consider the vault a complex relational database structure that is used to organize the data into a kind of graph build from Hubs, Satellites, and Links. Creating and modifying data vaults was a common task on many of my projects. While this example can be considered a *technical* DSL used by specialists responsible for loading the data and initially, setting up the vaults was the domain of the modeling team, over time the SMEs started to propose the structures and talk in terms of the DSL when talking about the data they wanted to access. We created a DSL that allowed creation or change of a vault via a series of one line statements, e.g., the following as some simple examples that set up a hub and then does a data quality check to verify it worked:

![hub instruction](./images/hub-instruction.png)

![dq instruction](./images/dq-instruction.png)

In this case we didn't generate raw dbt configurations. There are tools available to simplify the setup, e.g., the dbt vault extension [AutomateDV](https://automate-dv.com/)[^dbt-vault]. Such a tool can be considered a generic technical DSL layer on top of dbt. A DSL was needed here because even using this extension was too technical for our modeler users. Those modelers barely understood dbt. Using the extension also took too much effort to use and test even for a dbt expert.

- Data Mapping and Transformation (a.k.a. ETL) - In a project that loaded and converted clinical trial data from arbitrary input formats to an industry standard format. There are SMEs who do this as their primary job and they need to do custom versions of it for every clinical trial project and then deal with a series of changes. We implemented a language to express the mappings and transformations where SQL was only used in very special cases. An example of the kinds of high-level domain specific instructions are those for processing data from laboratory tests which converts multiple Laboratory values in a horizontal data layout, pivots it to be vertical as required by the standard and automatically deals with standard conversion tables and normal range checking. While you may not understand the details of this, describing how to do this is a central part of the ubiquotous language of clinical trail data. It typically requires detailed specifications that are then implemented as custom ETL or complex SQL. We implemented a single instruction, e.g., 

> `Lab Stack("WBC","WHITE BLOOD CELL COUNT","HEMATOLOGY","","BLOOD",LBHLAB,GEND,WBCRES,WBCU_)`

A clinical data conversion SME could read like a sentence because the parameters are in an expected order of the domain when working with this type of data. This is a simple single instruction we also have sequences of instructions that work as a unit, e.g., *Nesting* is the ability to use instructions within each other to provide seamless transformations while eliminating the need for temporary variables.


  [^data-vault]: See https://www.data-vault.co.uk/what-is-data-vault/ or google 'data vault' to see the massive amount of information available about it. 
  [^dbt-vault]: We used an earlier version of it called dbt-vault. It may have become easier to configure since then. What's important is the example of hiding  tecnical details from the SMEs by wrapping it with a DSL. An important benefit of a tool evolution like this, discussed later in this article, is that the DSL isolates you from the tech changes, see: TODO: write section on tech evolution. TODO: investigate whether it's more DSL in the new version.

Both of the above DSL examples were relatively simple languages, in part because their scope was relatively small and they were relatively technical. They were substantially easier to implemented because they were a layer on top of dbt. A tool like dbt is ideal for DSL creation because it is text based (a.k.a. configuration-as-code). The DSL then generates the dbt configuration files[^generate-more-than-dbt]. The focus can be on creating the language of the domain rather than that plus deep technical challenges related to making it possible to execute the DSL instructions. Next, I'll briefly describe another example that doesn't use dbt but covers a much richer domain at a much more domain specific level. 

Clinical trails are done to evaluate new medicines. They always start with writing a scientific specification of the evaluation called a *Clinical Protocol*. We built a DSL that enables the SMEs that specify data collection, calculations, workflows, and reports to be run on specialized clinical trial software systems. The following show examples of the IDE for defining these configurations.

The following shows an example of how a DSL can look like a form but still contain complex domain specific instructions. For example there are multiple expressions in the fields that reference data in other parts of the DSL, e.g., "First Scheduled" is defined as "Activation Completion + 6 days". These expressions can be arbitrary complex and the user is guided so that they only create valid expressions while still just typing. 

![form example](./images/visit-example.png)

The following shows a more complex DSL structure for defining when patients move through the phases of the clinical trial, e.g., they move when specific expressions about the eligibility critera evaluate to true. The implementation of this is essentially a state transition diagram. While we have the capability to build a pure state transition DSL, we instead used the language of the SMEs in the domain.

![disposition example](./images/disposition-example.png)

A DSL with this level of complexity requires more infrastructure than just a way to generate dbt files. In this case the runtime that needed to be configured is made up of many separate systems each different and each evolving what they support at different rates.  Luckily, there are powerful tools available for building DSLs of this complexity.[MPS] 

  [^MPS]: Getting into DSL technology requires a separate article. Some great places to start are looking at [MPS](TODO: get URL) and [LionWeb](TODO: get URL)

  [^generate-more-than-dbt]: It's never a simple as just generating dbt config files but that is the majority of what is generated. Other things like database DDL, blocks of shell scripts or python code are also generated to fill gaps between tools like dbt.

## APIs

The previous sections covered how dbt or an DSL that extends dbt would serve the Data APIs. We haven't talked about how to implement the regular API, e.g., http REST calls to retrieve data or do other processing.[operational-apis]  It is my believe that there is a deep problem with the current state of APIs and how clients use them, especially when we are trying for strong domain boundaries. APIs typically do one, rather restricted thing, e.g., retrieve some data possibly filtered, store some data, etc, launch some processing. Ideally the APIs match the part of the language of the domain that we want to expose to clients. Current technology doesn't allow an API to do the kind of rich semantic operations that the ubiquitous language supports. The client needs to string together API calls to do something like select some data, transform it, calculate something, format it, and bring back the right subset of the results. I'm not talking about just SQL statements. I'm talking about doing interesting things in the ubiquitous language. DSLs offer a novel way to define APIs that solve this problem as indicated in the following picture.

[operational-apis]: TODO: Investigate why data-mesh calls these 'operational APIs. The APIs labeled as 'O' in the diagrams. Operational sounds like they are limited to just managing the domain vs. accessing the data via them. Do they consider the 'D' APIs to be both the database access to data-products via SQL and the http style access?

![full DSL architecture](./images/full-dsl-architecture.png)

The API accepts statements in the domain language, executes them and returns the results. This has benefits including:
- The client gets to fully express the full set of semantic actions they want to perform in the language instead of a series of separate API calls
- The language is part of the domain boundary because clients can't do anything that the language doesn't support. Making traditional API calls allows more extensive data extraction and manipulation without these limits.
- Only one API that accepts the language need be implemented[^language-based-api-limits]

  [^language-based-api-limits]: Yes there will potentially need to be different APIs for different aspects or sub-sets of the language. I'm exaggerating for impact. 

As discussed in earlier sections, there are multiple levels of language when using dbt. The out-of-the-box, the addition of macros, and the addition of the semantic layer. Each can be exposed a data API that builds as the implementation of the data mesh evolves. The full DSL-based API would use all of these lower level languages. 

## Data Mesh DSL

If we model the language of the domains of our data mesh we will move along the path presented, e.g,
- Use an out-of-the-box generic DSL style tool like the basics of dbt. 
- Expand to use more features of the tool, potentially in combination with other tools, e.g., use of the semantic layer language of dbt potentially in combination with another tool to do more advanced data quality checking, e.g, [Great-Expectations](TODO: get url)
- Introduce a DSL. Frequently the first DSL tends to be more technical
- Expand the DSL to be more targeted at the SMEs of the domain. (Ideally you'd skip the more technical and start your DSL work here.)
- Expand to more comprehensive DSLs covering different domains.

You need not create a unqiue DSL for every domain, especially in the early part of the journey. There are almost certainly common data structures and operations shared by domains and basic things like how you express your data-products could be supported by a common DSL. For example every data product likely needs common ways to express:
- Data quality constraints
- Retention
- Tests
- Access policies
- API characteristics

## Everyone wants Self-Service

Few have credibly attained *self-service* data and processing and there is little agreement on how attain it: low-code/no-code, drag-and-drop UIs, AI/ML, Citizen Data Scientists, etc. I define self-service as the ability of the users to create executable solutions in or from the domain without the IT team doing a software development cycle. The solution can be as simple as getting access to existing data and using it to create new data or as elaborate as building a new application. With any of the dbt intermediate architectures described above in place self-service of things beyond simple are fully enabled. With the full DSL in place we attain elaborate self-service. For example, a data analyst could:
- Define new data models inside the domain 
- Use those domains to create a new data product to expose to other analysts
- Use the internal or data product models to define a new metric and expose that

If we don't attain these levels of self service we will never break out of the cycle of always being behind the business demands. Even more important, the right solutions will be built because the business users won't make mistakes on what to build they way it so frequently happens in a standard IT software dev cycle.

If we allow business users to build their own solutions, it needs to be done at level equivalent to an IT solution. This means: 
- Testing - before it can be used in production the analyst built solution needs to be tested. Dbt includes test automation and data quality checking as part of its language. Part of building a real DSL must include either including and integrating the dbt testing features or, ideally, having domain specific ways to test.
- Governance - before it can be moved to production impacts must be understood and managed, versioning must be supported, updates to metadata documentation must be done. Dbt includes a promotion process the supports moving new solution elements from dev to production, it supports versioning (and major extensions to versioning are coming soon), documentation is automatically produced.

A full DSL typcially includes an integrated editing and testing tool, e.g., an IDE style tool that is specific to the DSL. This level of DSL support dramatically enhances the self service. 

##  DSL Inside vs. Outside the Domain

It may be necessary to formalized two kinds of ubiquitous languages:
- the language used to do the work inside the domain boundary
- the language used by clients to interact with the domain

The language inside the domain can express operating on all the internal capabilities and data. External clients may be much more restricted in what they can access or do. When focused on the data mesh you are most likely to start with the client language, e.g., how to they interact with the data products. 

## Summary 

# Appendix

## Bounded Domains is Data Mesh++
Write about how Data Mesh is too narrow a name vs. Bounded Domains. It's not just about the data. It needs to cover all the systems not just the new ones on a cloud platform dedicated to data access and analytics. It needs to be an Enterprise Architecture pattern. 

TODO: This is where we revisit the question about apps in the domain vs. just data mentioned earlier in the article.
