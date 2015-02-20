---
layout: post
title: "JS ad-hoc function factory"
date: 2014-01-25 00:26
comments: true
published: true
categories: 
---

A fun way to avoid repetition:

```javascript
(function addFileField(path, styles) {
  var properties = {};
  properties[path] = { styles: styles };
  eventSchema.plugin(attachments, {
    directory: en.pluralize(path),
    storage: { providerName: 'aws2js', options: config.S3 },
    properties: properties
  });

  return addFileField;
})

('avatar', {
  thumbnail:  {
    thumbnail: '120x120^',
    gravity: 'center',
    extent: '120x120'
  },
  retina_thumbnail:  {
    thumbnail: '240x240^',
    gravity: 'center',
    extent: '240x240'
  }
})

('coverPhoto', {
  original: {},
  fitted: {
    thumbnail: '800x300^',
    gravity: 'center',
    extent: '800x300'
  },
  retina_fitted: {
    thumbnail: '1600x600^',
    gravity: 'center',
    extent: '1600x600'
  }
});
```
