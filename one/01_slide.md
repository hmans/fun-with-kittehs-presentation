!SLIDE title-slide
# **FUN WITH KITTEHS**

!SLIDE center
![kitteh1](kitteh1.jpg)

!SLIDE center
![kitteh2](kitteh2.jpg)

!SLIDE center
![kitteh3](kitteh3.jpg)

!SLIDE
# Awwww. #

!SLIDE
# Let's build a site with kitteh pix! lol! #

!SLIDE
# and thumbnails! #

!SLIDE
# FUCK YEAH **THUMBNAILS**! #

!SLIDE center
![kitteh1](kitteh1_thumb.jpg)
![kitteh2](kitteh2_thumb.jpg)
![kitteh3](kitteh3_thumb.jpg)

!SLIDE
# Paperclip #

!SLIDE
    @@@ ruby
    class Kitteh < ActiveRecord::Base
      has_attached_file :picture, :styles => {
        :medium => "400x300>",
        :thumb => "200x200>"
      }
      
      # Defining view properties in
      # controller code? ORLY?
    end

!SLIDE
    @@@ ruby
    # Let's refresh a quadzillion
    # kitteh pix lol!
    
    rake paperclip:refresh
    
    # zzZz
    #       zZzzZ
    #                zzZzzZ
    #   zzzZzz

!SLIDE
# CarrierWave

!SLIDE
    @@@ ruby
    class PictureUploader < CW::Uploader::Base
      storage :file
    end
    
    class Kitteh
      mount_uploader :picture, PictureUploader
    end
    
    # What?

!SLIDE
    @@@ ruby
    class PictureUploader < CW::Uploader::Base
      include CarrierWave::RMagick

      process :resize_to_fit => [800, 800]

      version :thumb do
        process :resize_to_fill => [200,200]
      end
      
      # View properties in an upload
      # storage class? ORLYRLYRLY?
    end

!SLIDE
    @@@ ruby
    instance = PictureUploader.new
    instance.recreate_versions!

    # "This uses a naive approach which
    # will re-upload and process all versions."
    #
    #      ...zZzzzZzz...

!SLIDE
# STOP.

!SLIDE center
# Kitteh pix is SIRIUZ BIZNIZ.
![bizniz](bizniz.jpg)

!SLIDE
# Once we have millions of kitteh pix, we're fucked proper.

!SLIDE
# Like, Google would never buy us. Probably not even Yahoo.

!SLIDE
# srsly, this sucks.

!SLIDE
# kitteh pix should be... #

!SLIDE bullets
# ...thumbnailed on the fly! #
* so our developers can mess around with thumbnail sizes without running a migration on millions of pix.

!SLIDE bullets
# ...under control of the view layer! #
* srsly. Thumbnail sizes are a view matter.

!SLIDE center
# **DRAGONFLY.**
![dragonfly](unicorn.jpg)

!SLIDE
# Standalone

!SLIDE

    @@@ ruby
    > df = Dragonfly[:images]
    > df.store 'important data lol!'
     => "2011/05/11/14_03_38_403_file"

!SLIDE

    @@@ ruby
    > file = df.fetch
                "2011/05/11/14_03_38_403_file"
     => #<Dragonfly::Job:0x10375d2b8 ...> 

    > file.data
     => "important data lol!" 

    > file.url
     => "/media/BAhbOgZmILzf3rffwM19m...xaWxl" 

    # URL, waitwhat?

!SLIDE
# Middleware, ZOMG! #

!SLIDE

    @@@ ruby
    config.middleware.insert_after 'Rack::Lock',
      'Dragonfly::Middleware',
      :images,  # Dragonfly instance to serve
      '/media'  # Base URL

!SLIDE bullets
# THUMBNAILS! #
* (and other transformations.)

!SLIDE

    @@@ ruby
    > df.store File.new('kitteh.jpg')
     => "2011/05/11/14_15_29_375_kitteh.jpg" 

!SLIDE

    @@@ ruby
    > img = df.fetch
          "2011/05/11/14_15_29_375_kitteh.jpg"
     => #<Dragonfly::Job:0x00000103df6f98 ...> 

    > img.url
     => "/media/BAhbBlsHOgZmSSIoMjAxMS...FVA"

!SLIDE

    @@@ ruby
    > img.thumb('200x200')
     => #<Dragonfly::Job:0x00000103dac560 ...> 
     
    > img.thumb('200x200').url
     => "/media/BA6d7shbB1svffIoMjAxMS8...wZU" 

!SLIDE

    @@@ ruby
    > img.thumb('200x200').watermark.url
     => "/media/BA6d7s7s8ssvffIo123dMS8...fxe" 

!SLIDE bullets
# Dragonfly middleware will... #
* fetch file from disk/S3/whereever
* apply transformations
* and serve the result.

!SLIDE
# What about ActiveModel? #

!SLIDE

    @@@ ruby
    class Kitteh < ActiveRecord::Base
      image_accessor :picture
      # assumes picture_uid (string)
    end
    
    @kitteh.picture = File.new('kitteh.jpg')

    # also works in file upload forms
    # derp derp derp

!SLIDE
# show.html.haml

    @@@ ruby
    = image_tag @kitteh.picture
                  .thumb('200x200').url

!SLIDE bullets
# Caching is your duty
* Rack::Cache, Varnish, ...

!SLIDE bullets
# ...or serve everything off S3
* Useful on Heroku, since its Varnish isn't really all that awesome

!SLIDE

    @@@ ruby
    app = Dragonfly[:images].configure do |c|
      c.datastore =
        Dragonfly::DataStorage::S3DataStore.new

      c.datastore.configure do |d|
        # ... S3 configuration ...
      end

      c.define_url do |app, job, opts|
        Thumb.url_for_job(job)
        # Thumb also takes care of
        # S3 expiry.
      end
    end

!SLIDE bullets
* (probably coming soon as a Dragonfly plugin)

!SLIDE bullets
# The end.
* Any questions?