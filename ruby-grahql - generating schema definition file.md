While developing your GraphQL api using [graphql-ruby](https://graphql-ruby.org/) you will probably come across the need of generating the schema definition file from the schema you defined in your code, so other front-end client would be able to consume it, or generate types automatically based on that schema.

There is this [github issue](https://github.com/rmosolgo/graphql-ruby/issues/2501), where [this comment](https://github.com/rmosolgo/graphql-ruby/issues/2501#issuecomment-536199861) outlines how you can accomplish the code generation. I couldn't make the same example work in my code so I came up with another solution.

It will also involve what was implemented in [this example](https://github.com/howtographql/graphql-ruby/blob/master/lib/tasks/graphql.rake) from https://howtographql.com.

We'll define a custom rake task and a rake task from the ruby-graphql package.

Start by creating the file `lib/taks/graphql.rake`. 

The file itself will be really simple:

```ruby
require "graphql/rake_task"
require "rake"

GraphQL::RakeTask.new(
  load_schema: -> (_task) {
    require_relative '../../app/graphql/dcms_schema'
    DcmsSchema
  }
)

namespace :graphql do
  task export: :environment do
    Rake::Task["graphql:schema:dump"].invoke
  end
end
```



So, first we are requiring `graphql/rake_task` so we can declare a graphql rake task and require `rake` to be able to invoke a rake task from within the file.

Here we declare the rake task called `graphql:schema:dump`. Inside the `load_schema` callback we require our schema definition file and return the schema class at the end of the callback.

```ruby
GraphQL::RakeTask.new(
  load_schema: -> (_task) {
    require_relative '../../app/graphql/dcms_schema'
    DcmsSchema
  }
)
```



After declaring the graphql rake task, we create our custom task by doing the following:

```ruby
namespace :graphql do
  task export: :environment do
    Rake::Task["graphql:schema:dump"].invoke
  end
end
```

Inside it, we are simply calling the rake task we previously defined with `Rake::Task["graphql:schema:dump"].invoke` 

Now, in the terminal we are able to execute our rake task by typing the following

```:bookmark_tabs:
$ rake graphql:export
```

If all works, you should be able to see two files: `schema.graphql` and `schema.json` in the root of your project.

You can check the rake tasks documentation for graphql-ruby here: https://graphql-ruby.org/api-doc/1.9.12/GraphQL/RakeTask