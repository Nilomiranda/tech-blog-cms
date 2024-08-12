–––
year: 2023
published: true
description: Learn what you need to setup image upload with Cloudinary and Rails
–––


This guide will walk you through the steps to setup simple image uploading in Rails using cloudinary service and ActiveStorage.

You can reference all documentation here:

[Ruby-on-Rails SDK– Upload + Image, Video Transformations | Cloudinary](https://cloudinary.com/documentation/rails_integration#installation)

[Active Storage integration | Cloudinary](https://cloudinary.com/documentation/rails_activestorage#active_storage_configuration)

[Active Storage Overview — Ruby on Rails Guides](https://edgeguides.rubyonrails.org/active_storage_overview.html)



## Installing Cloudinary's SDK

Let's start by installing Cloudinary's SDK for rails:

In your `Gemfile` insert the following line:

```
gem 'cloudinary'
```

And then run `bundle`.



## Setting up your Cloudinary credentials

After installing, you'll need to setup all needed credentials and other useful variables that will be sent in each upload request to cloudinary apis.

There's a few options, and the one we are going to use here is by creating a file inside the `config` folder called `cloudinary.yml`

You can download a pre-customized using this link: [cloudinary.yml](https://cloudinary.com/documentation/rails_integration#configuring_in_the_cloudinary_yml_file)

If you couldn't download the file, you need this basic set of configuration to get you started:

```yaml
development:
  cloud_name: <%= ENV['CLOUDINARY_CLOUD_NAME'] %>
  api_key: <%= ENV['CLOUDINARY_API_KEY'] %>
  api_secret: <%= ENV['CLOUDINARY_API_SECRET'] %>
  enhance_image_tag: true
  static_file_support: false
```

> It's good to make use of environment variables to put your credentials in this file. You can easily manage env variables using the [dotenv gem](https://github.com/bkeepers/dotenv)



## Setting up Rails Active Storage

Setting up Active Storage is fairly simple, but you'll need to have either [ImageMagick](https://imagemagick.org/index.php) or [libvips](https://github.com/libvips/libvips) installed since Rails relies on one of these libs for image analysis and transformations.

> If you're gonna handle image/video or PDF previews, you'll also need other libs.
>
> Your can see the full list of requirements [here](https://edgeguides.rubyonrails.org/active_storage_overview.html#requirements).



In order to work properly, active storage will need to work with three tables in your application: `active_storage_blobs`, `active_storage_variant_records` and `active_storage_attachments`.

Don't worry cause you don't need to manually create and set them up. Just run `rails active_storage:install` to generate the migration file and as always, run `rail db:migrate`.



There's some [additional notes](https://edgeguides.rubyonrails.org/active_storage_overview.html#setup) about how this tables work and how they will be used across your application. It's worth taking a look, specially if **you use UUIDs instead of integers as primary keys**.



## Setting Up Cloudinary with Active Storage



Now we are ready to setup Cloudinary to be used by Active Storage.

Head to your `config/storage.yml` to create a cloudinary service.

```yaml
cloudinary:
  service: Cloudinary
  folder: MEDIA_FOLDER_NAME_YOU_CREATED_IN_CLOUDINARY
```

> Don't forget to change **MEDIA_FOLDER_NAME_YOU_CREATED_IN_CLOUDINARY** with a valid folder name you setup in your cloudinary dashboard.



Now let's tell Active Storage that we want to use this cloudinary service.

In `config/environments/development.rb` put the following inline:

```ruby
config.active_storage.service = :cloudinary
```



> Note: This line is probably already declared in this file as `config.active_storage.service = :local` , so you just to replace the assignment with `cloudinary` .



## Associating image file with application Model

Let's say that our application has a `Publication` entity that the user can create with a `thumbnail` image.

Add `has_one_attached :thumbnail` to your `publication.rb` file.

> This is a case where the entity has ONE file associated with a column.
>
> If you need to storage and handle multiple images take a look [has_many_attached](https://edgeguides.rubyonrails.org/active_storage_overview.html#has-many-attached)

In your view file, inside your form, create a simple file input:

```erb
<%= form.file_field :thumbnail %>
```



Now, inside your controller, that handles the form submission, you can do the following:

```ruby
@publication = News.create params # any other param your entity may need

@publication.thumbnail.attach(params[:thumbnail])
```



And you can render your image like this:

```erb
<img src="<%= publication.thumbnail.attachment.url %>" alt="A nice alt value" />
```



Here are more useful references to work with model attachments:

[ActiveStorage::Blob (rubyonrails.org)](https://api.rubyonrails.org/classes/ActiveStorage/Blob.html)

[ActiveStorage::Attachment (rubyonrails.org)](https://api.rubyonrails.org/classes/ActiveStorage/Attachment.html)

[ActiveStorage::Attached::One (rubyonrails.org)](https://api.rubyonrails.org/v6.1.3/classes/ActiveStorage/Attached/One.html)
