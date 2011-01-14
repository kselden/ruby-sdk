# transloadit

Fantastic file uploading for your web application

## Description

This is the official Ruby gem for the [Transloadit](transloadit.com) API. You
can use it to upload files to the REST API and to ease the creation of
Transloadit-enabled upload forms.

## Install

    gem install transloadit

## Getting started

To get started, you need to require the 'transloadit' gem:

    $ irb -rubygems
    >> require 'transloadit'
    => true

Then create a Transloadit instance, which will maintain your authentication
credentials and allow us to make requests to the API.

    transloadit = Transloadit.new(
      :key    => 'transloadit-auth-key',
      :secret => 'transloadit-auth-secret'
    )

### 1. Resize and store an image

This example demonstrates how you can create an assembly to resize an image
and store the result on [Amazon S3](http://aws.amazon.com/s3/).

First, we create two robots: one to resize the image to 320x240, and another
to store the image in our S3 bucket.

    resize = transloadit.robot '/image/resize',
      width:  320,
      height: 240

    store  = transloadit.robot '/s3/store',
      key:    'aws-access-key-id',
      secret: 'aws-secret-access-key
      bucket: 'bucket-name'

Now that we have the robots, we create an assembly (which is just a request to
process a file or set of files) and let Transloadit do the rest.

    assembly = transloadit.assembly open('image.jpg'),
      :steps => [ resize, store ]

When the `assembly` method returns, the file has been uploaded but may not yet
be done processing. We can use the returned object to check if processing has
completed, or examine other attributes of the request.

    # returns the unique API ID of the assembly
    assembly.id # => '9bd733a...'
    
    # returns the API URL endpoint for the assembly
    assembly.url # => 'http://api2.vivian.transloadit.com/assemblies/9bd733a...'
    
    # checks if all processing has been completed
    assembly.completed? # => false
    
    # cancels further processing on the assembly
    assembly.cancel! # => true
    
    # checks how many bytes were expected / received by transloadit
    assembly.bytes_expected # => 92933
    assembly.bytes_received # => 92933
    
It's important to note that none of these queries are "live" (with the
exception of the `cancel!` method). They all check the response given by the
API at the time the assembly was created. You have to explicitly ask the
assembly to reload its results from the API.

    assembly.reload!

### 2. Uploading multiple images

    Multiple files can be given to the `assembly` method in order to upload more
    than one file in the same request. You can pass a single robot for the
    `steps` parameter, without having to wrap it in an Array.

        assembly = transloadit.assembly(
          open('puppies.jpg'),
          open('kittens.jpg'),
          open('ferrets.jpg'),
          :steps => store
        )