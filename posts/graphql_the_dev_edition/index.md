
# GraphQL the Dev Edition
![title-image](https://raw.githubusercontent.com/mthomps4/posts/master/posts/graphql_the_dev_edition/images/luke-chesser-JKUTrJ4vK00-unsplash.jpg)

Photo by [Luke Chesser](https://unsplash.com/@lukechesser) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

# **Intro**

It’s difficult to be a developer and not experience the hype around GraphQL. I’ve spent the last three years implementing GraphQL APIs in Elixir and Node, watching the tooling and community grow along the way. In this post, I want to shed some light on the pros and cons, breakdown some of the packages used and leave you and your team with enough insight to make an informed decision on GraphQL.

*Note: Your GraphQL server does not have to be implemented with Node. However, I will be detailing a bit of GraphQL history with JavaScript in mind as most will be interacting with these tools at some point via the client. You can see a full list of supported server languages at [graphql.org/code](http://graphql.org/code).*

If you would like to skip the history and packages and read the pro and con section scroll down to *GraphQL the good and bad.* Sadly, I could not get anchor tags to work today in Medium. ¯\_(ツ)_/¯

# **What’s the Hype all about?**

> GraphQL is an open-source data query and manipulation language for APIs, and a runtime for fulfilling queries with existing data.

As is with any new shiny web tool we’ll start with the docs.

The documentation found at [graphql.org](http://graphql.org/) does a good job at defining what GraphQL is and isn’t, so I’m not going to veer too far from that here. Navigating to [graphql.org](http://graphql.org/) you’ll arrive at a landing page that gives a quick three bullet response to the most common pros.

- Describe your data
- Ask for what you want
- Get predictable results

We will talk about these points more in a moment, but for a lot of lead developers, we have the responsibility to research these tools to ensure their merit. The last thing you want is to see your team ramp up on a new open-sourced tool only to have it crumble from underneath us a few months later. With the open-sourced community being well, open-sourced, it is easy to have a pessimistic approach to development and the tools we use, or at least I do. My internal conversations start with something like, “That all sounds great, but everyone hypes up their work. How do I know I can trust this one?”

**History of GraphQL**Let’s take a quick dive into how it all started and where a few packages are now.

Through Google searching around you’ll find that GraphQL started by Facebook as an internal tool back in 2012. Facebook had an issue of building a solid API for their mobile apps that could consistently describe their data as they scaled. You can read more of their release details [here](https://engineering.fb.com/core-data/graphql-a-data-query-language/). As you can imagine, this took a while to refine. By 2015, Facebook had released this tool into the wild to help drive the process, similar to how React grew into what we know today. Fast forward a few more years, and GraphQL was moved from Facebook to the newly established [GraphQL Foundation](https://foundation.graphql.org/members/), hosted by [Linux Foundation](https://www.linuxfoundation.org/). Now you have a tool created by a tech giant, refined by the community, and supported by one of the largest open-source investors today. [Other companies](https://foundation.graphql.org/members/) have joined the GraphQL Foundation in support AWS, IBM, and Twitter being a few. If that’s not enough, a quick look into the JavaScript community shows the [graphql npm package](https://www.npmjs.com/package/graphql) alone was downloaded by over 2 million users last week. This is just one package within a single community. But again, GraphQL is not tied to JavaScript ([graphql.org/code](http://graphql.org/code)).

**History of GraphQL tools**

Great! GraphQL seems legit enough to look into further, but now I notice there’s a ton of tooling and ways to implement it.

Agreeably, the noise is hard to sort through. You search GraphQL and a ton of articles (like this one), packages, and tools all appear. There are even articles like this one [“13 GraphQL Tools and Libraries You Should Know in 2019”](https://blog.bitsrc.io/13-graphql-tools-and-libraries-you-should-know-in-2019-e4b9005f6fc2) that try to set expectations for what developers should know.

I’m going to highlight a few of them as a quick overview of where we are:

- Express GraphQL
- GraphQL Yoga
- Prisma (1 and 2)
- Apollo Server
- AWS Amplify

**NOTE:** I’d like to mention that ALL of these packages have merit. There is no one-size-fits-all GraphQL package. I suggest you explore the community, look around and find what best suits your needs. This is not meant to weigh one over the other but hopefully can help get you started.

**Express GraphQL**

This is probably the quickest way to demo and experiment with GraphQL in my opinion. This setup uses the ‘express-graphql’ and ‘graphql’ packages. This used to be the example given within the graphql docs itself. Ignoring some details below, you can see that Express sets up a route at `/graphql`. This exposes an endpoint pointing to our full `graphql` setup. From here, it's all back to the GraphQL docs to learn how to build out a new Schema, Queries, Resolvers, etc. There’s no doubt these two packages make up the foundation of some of the following tools. This setup will allow you to get your feet wet, spending more time in the GraphQL docs and less time questioning how to set it up.

    var express = require('express');var graphqlHTTP = require('express-graphql');var { buildSchema } = require('graphql');var schema = buildSchema(`  type Query {    hello: String  }`);var root = { hello: () => 'Hello world!' };var app = express();app.use('/graphql', graphqlHTTP({  schema: schema,  rootValue: root,  graphiql: true,}));app.listen(4000, () => console.log('Now browse to localhost:4000/graphql'));

As the community took off with GraphQL, you could imagine it became sort of a leg race to be *THE* GraphQL solve all. If you ask me — that marathon is continuing today. The remaining list below has grown and evolved since their first iterations. As a brief of each, I may have left something out, but again they each have merit, so if something strikes your interest, ***dive in***!

**[GraphQL Yoga](https://github.com/prisma/graphql-yoga)**

One of the first tools I remember seeing on the scene — GraphQL Yoga. A look into GraphQL Yoga and you’ll see that it is a collection of packages and tools brought together to make remembering all the boilerplate setup you may need easier. When it started, it used the express-graphql package above, but you’ll notice now that it uses one of the other tools mentioned here: Apollo Server. So, why not just use Apollo Server instead? Some may opt for just that. GraphQL Yoga becomes an abstraction with pros and cons. How much code do you wish to maintain vs having a tool do it for you? Will you run into a wall at some point? In my experience, using any magical create-thing-package will at some point lead to issues. However, for most common apps, you may be just fine. Use your judgment.

GraphQL Yoga’s Pitch:

> Using these packages individually incurs overhead in the setup process and requires you to write a lot of boilerplate. graphql-yoga abstracts away the initial complexity and required boilerplate and lets you get started quickly with a set of sensible defaults for your server configuration.graphql-yoga is like create-react-app for building GraphQL servers.

**[Prisma](https://www.prisma.io/)**

Prisma came on the scenes trying to solve the boilerplate issue. I already have my database, I already have my models, can’t we just wrap this in some way? Focusing more on Deployment options and wrappers, Prisma started creating the idea of GraphQL as a service tooling. This came with pre-built filters from your models ditching all custom boilerplate you may have needed otherwise. Below you’ll see their example of posts, `prisma.posts` already has a `where` and `orderBy` clause defined, but look further, `where` also has helpers for things like *`_contains`,* `_ends_with`, etc.

    const posts: Post[] = await prisma.posts({  where: {    published: true,    title_contains: "GraphQL"    author: {      email_ends_with: "@prisma.io"    }  },  orderBy: "createdAt_ASC"})

Prisma is now working on 2.0 which is to include its own type of ORM tooling expanding the above into a more chain-able dot syntax and expanding it’s database migration tools.

    const postsByUser: Post[] = await photon  .users  .findOne({ where: { email: 'ada@prisma.io' }})  .posts()

At this point, GraphQL as a service seems to be a big push. I’ll be interested to see how some of these tools shake out over time. Prisma learned a lot through its first iteration, and while the initial 1.0 setup felt a bit off, 2.0 seems to be making leaps based off community feedback. This is worth keeping an eye on.

**[AWS Amplify](https://aws-amplify.github.io/)**

Do you run all your services in the cloud already? Are you considering it? Do you want to jump on the Serverless train? AWS, like other tooling, brings their services together to provide what seems to be as close to GraphQL as a service as you can be. I believe as these tools continue to mature, we will see more of a “point this tool at your database…” type of setup. You can set up your DB of choice, add your Schema, Resolver logic, and let AWS manage the rest. Using AWS, which has its own learning curve, removes a lot of the overhead and boilerplate. You’ll be up and running quickly and back to focusing on solving your project’s problems, instead of configuring all the things. This also gives you the freedom to quickly tie into other AWS resources such as S3. Do you need to have an endpoint to upload files? Chances are you were probably going to use S3 anyway. Do you have IAM roles set? Use them with GraphQL for your authorization and between Amplify and AppSync you’ll be set in no time.

Do you want to stay up to date with AWS Amplify? I’d recommend giving [Nader Dabit](https://twitter.com/dabit3) a follow.

**Apollo Server**

I know I said there was no one-size-fits-all GraphQL package, I kind of lied. It’s hard to search GraphQL and *NOT* see Apollo mentioned at this point in the game and for a good reason. While AWS and others are pushing forward with serverless tech, Apollo, at least for now, has kind of set the standard for server/client setups. Regardless of which server-side setup you choose, you’ll be reaching for Apollo at some point for the client-side. Apollo client has been the most supported, and community-driven. It has also improved upon its cache store, subscriptions, and stayed up to date with ESX features and state management features like React Hooks. At this point, Apollo has enough tooling and configuration options to merit its own blog post. It’s safe to say if you aren’t sure which option is best for your team, Apollo is a safe bet.

# **GraphQL the good and bad**

# **Benefits**

- **One API Endpoint** — This is normally the first thing you hear about. Simplify all those REST routes for your user into one simple route. Example: `api/graphql` route.Granted you will be managing Queries and Mutations as “routes” but now the end-user can infer these from automated schema docs, GraphQL playground, etc.
- **Data Normalization** — Sometimes your API may manage calls and data that aren’t it’s own. This is always awkward for the consuming user. What if it errors? The user also has to parse this JSON blob and hope what they need is there. You can help the user by turning those third party calls into defined graphQL calls and normalize the return. This can be as simple as `{success: true, data: "JSON"}` or take this further, and define Types for that JSON blob. Now not only can the user quickly parse a successful call, but they can ask specifically for the information needed.

    getBlogPost {   success   data { id title body }   errors }

- **Multiple Languages** — If you haven’t caught on by now the community is huge. GraphQL has been around longer than most think and is more than just a fancy new way to build a web app. You can use GraphQL in just about any tech stack. *[graphql.org/code](http://graphql.org/code)*
- **Ease in** — Outside of a bit of boilerplate, you do not have to support your entire API all at once out of the gate. You can initialize your GraphQL server and migrate one call at a time, as needed. Supporting both your REST API and GraphQL API can be done simply by relaying calls between the two. There are also community works to wrap a REST API auto-magically with GraphQL. You don’t have to rush it. Take time to define your types and in the long run, it will be worth it.
- **Clean Data** — We’ve hinted at this already, but your data now becomes typed and defined. As long as you’ve got some error handling in play, you can normalize all your results into something human-readable and reliable. Adding things like custom `Scalars` enhances your data, and ensures it is serialized or parsed before ever reaching your DB/User.
- **Dynamic Use** — Once you get a few endpoints up and running, you’ll realize a lot of the functionality can be broken down into similar patterns. These patterns include filters, order, pagination, etc., as well as relational lookups via Resolvers. Not only will these patterns simplify maintenance for your devs, but now it’s on the end-user to ask for what they need. Instead of supporting 10 different routes for ‘users’, we simply have a query endpoint for `users` with all the required patterns in play. It's now on the end-user to decide what they need. Maintenance becomes adding new where filters with the appropriate joins as you scale, and the end-user is good to go. Your Front-End devs can also get used to this pattern. I need 'X' users with 'Y' posts. No more long conversations about which route does what, and worrying about if all the data returns. Ask and shall receive.

    users(where: {...}, orderBy: {...}, pagination: {...}) {   id   firstName   posts (where: {...}, orderBy: {...}) {     id     title     body    } }

- **Documentation** — Real talk: keeping documentation up to date is a pain. Tech moves fast. With tooling to auto-export Schema, GraphiQL, and [graphQL playground](https://www.prisma.io/blog/introducing-graphql-playground-f1e0a018f05d) at your fingertips, documentation is no longer an issue. Staying diligent to add a quick description line here and there and you are good to go. GraphiQl and Playground will pick up on these lines to give a description to the call, but because everything is now typed, they also show the defined input and output of said call. You may even opt to create a GraphQL playground for your external users/clients. Keep a live server up to date with your GraphQL schema that returns dummy data. Users/clients can use this playground to get a feel for your API and learn how to integrate on their own. With its schema documented, most questions can be answered without the back and forth phone calls.
- **Subscriptions** — Need real-time data? GraphQL supports subscriptions out of the box. Create your `Subscriptions` and have your FE peeps tie in.
- **Onboarding** — While I agree there is a lot of boilerplate to set up with GraphQL, once you are up and running, the patterns become super simple to replicate. Need a new Query endpoint? Anyone can look at previous examples and follow along. You may even use template utils to make this process even simpler. New team member? Have them interact with the GraphQL playground for a day. This will introduce business terminology, relations, and current use cases. With GraphQL your goal should be to follow consistent patterns for your end-users to take hold. Once you understand THE pattern, you’re good to go on creating new types and maintaining older DB relations.
- **Customization** — It is worth noting again, use GraphQL for what you need, how you need, with the tools you need.

# **Counter Weights (Cons)**

- **Boilerplate** — I agree that there is some overhead to get up and running with GraphQL. You’ll want to choose what seems to work best for your team and press forward. While you can slowly build up your API, remember there is a Front-End overhead here as well. Your team will probably want/need to set up some sort of cache/store with Apollo to make these calls easier moving forward. When adding new endpoints, it’s all about the pattern. It does become rinse and repeat, but there is still a chunk to replicate.
- **Learning Curve** — There is a bit of a learning curve to GraphQL. What’s a Mutation? What is a Resolver? How do I set up this the pattern I keep hearing about? Above, I mentioned that onboarding becomes easier to follow due to patterns in play. While that is true, your team will still need to learn, communicate, and establish this pattern. That takes some time.
- **Momentum** — Continuing on the note above, establishing the patterns and benefits take time. The time benefits from GraphQL become visible later in the game. Your team is in the thick of your production app and new features and maintenance are occurring. In my opinion, GraphQL is a long term investment. Over time you will refine this build, adding helpful utils, solidifying filters, and relational lookups. As you scale, your team will become more efficient in building with GraphQL and begin to enjoy the benefits as logic changes are needed. As an agency, sometimes this one can be frustrating. We build for our clients with the future in mind. Their team will benefit from the patterns and momentum that we created. We consider it our job to get the stone rolling.

# **Don’t recreate the wheel.**

While this may be obvious, it’s worth noting. GraphQL has the support of a rallying community. Don’t recreate the wheel before checking out the open-sourced world for solutions. You’ll spend a lot of time building out GraphQL boilerplate that could have been a simple import. Below are some packages that come to mind:

**Date Scalars**

- Node:[https://www.npmjs.com/package/graphql-scalars](https://www.npmjs.com/package/graphql-scalars)[https://www.npmjs.com/package/graphql-iso-date](https://www.npmjs.com/package/graphql-iso-date)
- Elixir:[https://github.com/absinthe-graphql/absinthe/wiki/Scalar-Recipe](https://github.com/absinthe-graphql/absinthe/wiki/Scalar-Recipe)[https://github.com/absinthe-graphql/absinthe/blob/master/lib/absinthe/type/custom.ex](https://github.com/absinthe-graphql/absinthe/blob/master/lib/absinthe/type/custom.ex)

**Dataloader:**

- Node: [https://www.npmjs.com/package/dataloader](https://www.npmjs.com/package/dataloader)
- Elixir: [https://github.com/absinthe-graphql/dataloader](https://github.com/absinthe-graphql/dataloader)
- Python: [https://github.com/graphql-python/graphene](https://github.com/graphql-python/graphene)

**[Code Generating Typescript Types](https://github.com/dotansimha/graphql-code-generator)**

# **Is GraphQL a good fit?!**

I usually go by a quick smoke test to determine if GraphQL is worth it.

**NO**

