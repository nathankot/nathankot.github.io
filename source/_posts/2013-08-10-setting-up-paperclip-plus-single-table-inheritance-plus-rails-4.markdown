---
layout: post
title: "Setting up Paperclip + Single Table Inheritance + Rails 4"
date: 2013-08-10 18:11
comments: true
categories: rails
---

If you want to have a single table describing a file upload, with different validations
for each file type this is a good way to set it up.

``` ruby attachment.rb

class Attachment < ActiveRecord::Base

  # Load up all descendant models

  @@descendants = Dir[
    Rails.root + 'app/models/**/*_attachment.rb'
  ].map do |f|
    File.basename(f, '.*').camelize
  end

  ###

  has_attached_file :file
  belongs_to :attachable, polymorphic: true

  before_create do
    break if @@descendants.each do |descendant|
      descendant = descendant.constantize
      other = self.becomes descendant
      if other.valid?
        next self.type = descendant.to_s
      end
    end

    throw ActiveRecord::RecordInvalid.new(
      "Uploaded file does not satisfy any available attachment types."
    )
  end

end

```


``` ruby image_attachment.rb

class ImageAttachment < Attachment

  validates_attachment :file,
    presence: true,
    size: { in: 0..5.megabytes },
    content_type: { content_type: /image/ }

end

```


``` ruby document_attachment.rb

class DocumentAttachment < Attachment

  validates_attachment :file,
    presence: true,
    size: { in: 0..5.megabytes },
    content_type: { content_type: [
      "application/pdf",
      "text/plain"
    ] }

end

```

So when you run something like `@attachment.save`, it will automatically choose the
best model for you and save it with that type :)
