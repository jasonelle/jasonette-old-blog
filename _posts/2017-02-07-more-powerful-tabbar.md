---
layout: post
title: Supercharged tab bar!
---

The [bottom tab bar](http://docs.jasonette.com/document/#tabs) is useful for switching between multiple views.

<img class='bordered medium' src='http://docs.jasonette.com/images/footer_tabs.jpeg'>

To make this work you would simply specify the `url` attribute for each tab, like this:

```json
{
  "$jason": {
    "body": {
      "footer": {
        "tabs": {
          "items": [{
            "text": "Tab 1",
            "url": "https://jasonbase.com/things/3fn.json"
          }, {
            "text": "Tab 2",
            "url": "https://jasonbase.com/things/2fn.json"
          }, {
            "text": "Tab 3",
            "url": "https://jasonbase.com/things/9fn.json"
          }]
        }
      }
    }
  }
}
```

This is sufficient for most use cases, but sometimes you may want more. For example you may want to:

1. Open a new view in a modal when you tap a tab item.
2. Open a browser when you tap a tab item.
2. Exectue an action (Such as open a camera) when you tap.

Until today these haven't been possible, since the only supported attribute was `url`.

However, this changes today. You can attach not only `url` but also `href` as well as `action`.

<img class='bordered medium' src='/assets/supertab.gif'>

Go download the `develop` branch ([iOS](https://github.com/Jasonette/JASONETTE-iOS) and [Android](https://github.com/Jasonette/JASONETTE-Android)) and try it out. Here's what has changed:

## href
You can now set `href` as well as `url`. `href` is obviously more powerful since you can specify not just the url but also the `view` or the type of `transition`. See below example:

```json
{
  "$jason": {
    "body": {
      "footer": {
        "tabs": {
          "items": [{
            "text": "Tab 1",
            "url": "https://jasonbase.com/things/3fn.json"
          }, {
            "text": "Tab 2",
            "href": {
              "url": "https://jasonbase.com/things/2fn.json",
              "transition": "modal"
            }
          }, {
            "text": "Tab 3",
            "href": {
              "url": "https://www.google.com",
              "view": "web"
            }
          }]
        }
      }
    }
  }
}
```

1. `tab 2` now opens in a modal view when you tap, instead of opening within the current view.
2. `tab 3` now opens a browser since it's a `"view": "web"` type href.


## action

You can even set an `action` attribute for a tab. You may see something like this in photo apps like Instagram--you tap one of the tab bar items and it opens a camera. Here's an example:

```json
{
  "$jason": {
    "body": {
      "footer": {
        "tabs": {
          "items": [{
            "text": "Tab 1",
            "url": "https://jasonbase.com/things/3fn.json"
          }, {
            "text": "Tab 2",
            "action": {
              "type": "$media.camera"
            }
          }, {
            "text": "Tab 3",
            "href": {
              "url": "https://www.google.com",
              "view": "web"
            }
          }]
        }
      }
    }
  }
}
```

`Tab 2` now runs the [$media.camera action](http://docs.jasonette.com/actions/#mediacamera) when you tap.

Note that you can attach **any** kind of actions here, not just camera.


## Tip

Sometimes people are confused about how the tabs work. Here's the rule of thumb:

**You MUST specify the tabs for every view.**

Let's say you have tab1, tab2, and tab3. All of those views MUST contain the **same** `tabs` object. If you only have the `tabs` object on `tab1`'s markup, the tab bar will go away when you navigate away to tab2 or tab3.

## Try it out!
As of today this works on both [Android](https://github.com/Jasonette/JASONETTE-Android) and [iOS](https://github.com/Jasonette/JASONETTE-iOS)
