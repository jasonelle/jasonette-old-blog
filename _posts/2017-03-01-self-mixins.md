---
layout: post
title: Self Mix-in
image: assets/mixins.jpg
---

In the last post, I've talked about [mix-ins](), which lets you mix in remote JSON objects to generate a new JSON object.

Well, today I have a powerful piece of dark magic which takes this to a whole new level.

**Meet Self-Mixin -- A mix-in that mixes a JSON object into itself**.

![self mixin](/assets/snake.png)

# Syntax

There's only one thing you need to know: `$document`.

`$document` refers to the **the currently loaded JSON tree**.

For example, let's imagine we've loaded the following JSON on Jasonette.

```
{
  "items": [{
    "type": "label",
    "text": "Label 1"
  }, {
    "type": "label",
    "text": "Label 2"
  }, {
    "type": "label",
    "text": "Label 3"
  }],
  "$jason": {
    "body": {
      "sections": [{
        "@": "$document.items"
      }]
    }
  }
}
```

Notice the `$document.items` part. This is a self mixin in action.

When resolving, Jasonette looks at the loaded JSON tree itself (`$document`) and searches for an attribute called `items`, then it replaces the mixin expression with the object, which results in:

```
{
  "items": [{
    "type": "label",
    "text": "Label 1"
  }, {
    "type": "label",
    "text": "Label 2"
  }, {
    "type": "label",
    "text": "Label 3"
  }],
  "$jason": {
    "body": {
      "sections": [{
        "items": [{
          "type": "label",
          "text": "Label 1"
        }, {
          "type": "label",
          "text": "Label 2"
        }, {
          "type": "label",
          "text": "Label 3"
        }]
      }]
    }
  }
}
```

<br>

# $document is contextual

`$document` is a contextual concept. The same `$document` reference can point to a different object depending on how the JSON object was loaded (**Was it loaded as the main JSON markup?** vs. **Was it loaded from the main markup as a mix-in?**)

Let's take a look at some examples:

## 1. Loaded as the main JSON markup

Here's a basic case:

**https://blah.blah/child.json**

```
{
  "$jason": {
    "head": {
      "title": "Test",
      "templates": {
        "body": {
          "sections": [{
            "items": [{
              "@": "$document.model"
            }]
          }]
        }
      }
    }
  },
  "model": {
    "type": "label",
    "text": "Bye World"
  }
}
```

When Jasonette loads and resolves the `"@": "$document.model"`, it takes the current JSON tree's `model` attribute and replaces it into the mixin expression, and we end up with:

```
{
  "$jason": {
    "head": {
      "title": "Test",
      "templates": {
        "body": {
          "sections": [{
            "items": [{
              "type": "label",
              "text": "Bye World"
            }]
          }]
        }
      }
    }
  },
  "model": {
    "type": "label",
    "text": "Bye World"
  }
}
```

But we already knew that. Nothing special here.

<br>

## 2. Loaded as a mix-in

This time, let's try taking the same `child.json` and mix it into another JSON called `parent.json` like this:

**https://blah.blah/parent.json**

```
{
  "@": "https://blah.blah/child.json",
  "model": {
    "type": "label",
    "text": "Hello World"
  }
}
```

Here's what will happen:

**Step 1.** Jasonette loads `parent.json` and discovers that there's a mixin to resolve.

**Step 2.** So it mixes in the contents of `https://blah.blah/child.json`. To refresh memory, `child.json` looks like this:

```
{
  "$jason": {
    "head": {
      "title": "Test",
      "templates": {
        "body": {
          "sections": [{
            "items": [{
              "@": "$document.model"
            }]
          }]
        }
      }
    }
  },
  "model": {
    "type": "label",
    "text": "Bye World"
  }
}
```

**Step 3.** Notice how `parent.json` and `child.json` both have the `model` attribute, but with different values. Whenever there's a conflict, the remotely mixed-in attribute (`model` from `child.json`) gets overridden by the main one (`model` attribute from `parent.json`) like this:

```
{
  "$jason": {
    "head": {
      "title": "Test",
      "templates": {
        "body": {
          "sections": [{
            "items": [{
              "@": "$document.model"
            }]
          }]
        }
      }
    }
  },
  "model": {
    "type": "label",
    "text": "Hello World"
  }
}
```

**Step 4.** Now Jasonette finds that there's still another mixin to resolve: `$document.model`. So it mixes in the `model` attribute.



The final result is:

```
{
  "$jason": {
    "head": {
      "title": "Test",
      "templates": {
        "body": {
          "sections": [{
            "items": [{
              "type": "label",
              "text": "Hello World"
            }]
          }]
        }
      }
    }
  },
  "model": {
    "type": "label",
    "text": "Hello World"
  }
}
```

> **Important distinction:** In the previous example the `$document.model` inside `child.json` was interpreted as `child.json` itself, but this time the same `$document.model` was interpreted as `parent.json`, since its interpretation was carried out **after** it was mixed into `parent.json`.

<br>

# Impossible is Nothing

Let's step back and think about what all this means.

This means now you can practically **manipulate ANY part of a JSON tree** at your will.

## 1. Organize the JSON any way you want

Previously Jasonette had some conventions you needed to follow. For example, you place templates under `$jason.head.templates`, data under `$jason.head.data`, actions under `$jason.head.actions`, and so forth.


{% raw %}
```
{
  "$jason": {
    "head": {
      "data": {
        "dwarfs": [{
          "username": "Doc"
        }, {
          "username": "Dopey"
        }, {
          "username": "Bashful"
        }, {
          "username": "Grumpy"
        }, {
          "username": "Sneezy"
        }, {
          "username": "Sleepy"
        }, {
          "username": "Happy"
        }]
      },
      "templates": {
        "body": {
          "sections": [{
            "items": {
              "{{#each model.dwarfs}}": {
                "type": "label",
                "text": "{{username}}"
              }
            }
          }]
        }
      }
    }
  }
}
```
{% endraw %}

This still applies, but you are no longer locked into writing your markup strictly this way since you now have access to ANY part of the entire JSON tree.

Here's an example that does the same thing as above:

{% raw %}
```
{
  "$jason": {
    "head": {
      "data": {
        "@": "$document.model"
      },
      "templates": {
        "@": "$document.view.body"
      }
    }
  },
  "view": {
    "body": {
      "sections": [{
        "items": {
          "{{#each model.dwarfs}}": {
            "type": "label",
            "text": "{{username}}"
          }
        }
      }]
    }
  },
  "model": {
    "dwarfs": [{
      "username": "Doc"
    }, {
      "username": "Dopey"
    }, {
      "username": "Bashful"
    }, {
      "username": "Grumpy"
    }, {
      "username": "Sneezy"
    }, {
      "username": "Sleepy"
    }, {
      "username": "Happy"
    }]
  }
}
```
{% endraw %}

Combine this with remote mix-ins and you have something like:

{% raw %}
```
{
  "$jason": {
    "head": {
      "data": {
        "@": "$document.model"
      },
      "templates": {
        "@": "$document.view.body"
      }
    }
  },
  "view": {
    "@": "https://blah.blah/body.json"
  },
  "model": {
    "@": "https://blah.blah/dwarfs.json"
  }
}
```
{% endraw %}

You can even mix-in the `$jason` object from remote:

{% raw %}
```
{
  "@": "https://blah.blah/jason_skeleton.json",
  "view": {
    "@": "https://blah.blah/body.json"
  },
  "model": {
    "@": "https://blah.blah/dwarfs.json"
  }
}
```
{% endraw %}


where `jason_skeleton.json` looks like this:

**https://blah.blah/jason_skeleton.json**

```
{
  "$jason": {
    "head": {
      "data": {
        "@": "$document.model"
      },
      "templates": {
        "@": "$document.view.body"
      }
    }
  }
}
```


<br>

## 2. Modularize and reuse

Once you get the hang of it, you'll come up with all kinds of creative ways of utilizing this feature. Let me explain one.

Let's go back to the dwarfs app example. What if we wanted to take the same app and just switch out the data to build a new app? Here's the original view:

**https://blah.blah/view.json**

{% raw %}
```
{
  "body": {
    "sections": [{
      "items": {
        "{{#each model.dwarfs}}": {
          "type": "label",
          "text": "{{username}}"
        }
      }
    }]
  }
}
```

As you can see, the template makes two assumptions:

1. `$jason.head.data.model` has an array called `dwarfs`
2. The `dwarfs` array contains objects with the attribute `username`. 

So we immediately run into a problem. This view is not really reusable outside of this dwarfs app context. For example we may have an API that returns the following:

```
{
  "response": {
    "movies": [{
      "title": "American Psycho"
    }, {
      "title": "Fight Club"
    }, {
      "title": "Matrix"
    }, {
      "title": "The Shining"
    }]
  }
}
```

First, let's fix the `model.dwarfs` part. Let's switch it out with something that's more generic: `model.items`

```
{
  "body": {
    "sections": [{
      "items": {
        "{{#each model.items}}": {
          "type": "label",
          "text": "{{username}}"
        }
      }
    }]
  }
}
```

Now let's go back to the main JSON. Currently it looks like this:

```
{
  "@": "https://blah.blah/jason_skeleton.json",
  "view": {
    "@": "https://blah.blah/body.json"
  },
  "model": {
    "@": "https://blah.blah/dwarfs.json"
  }
}
```
Let's change this since it wouldn't work anymore now that the view is expecting `model.items`. We need to create a mapping that transforms `model.dwarfs` into `model.items`, like this:

```
{
  "@": "https://blah.blah/jason_skeleton.json",
  "view": {
    "@": "https://blah.blah/body.json"
  },
  "model": {
    "items": {
      "@": "dwarfs@https://blah.blah/dwarfs.json"
    }
  }
}
```

Here we have used partial mixin (introduced in [the last article on mixins]()) to select `dwarfs` and assign it to `$document.model.items`.



This solves the first problem.

<br>

Now, we still have one more problem to solve. We want to make the `"text": "{{username}}"` part generic as well so it can be reused for any other cases. How can we create a mapping for this?

Let's go back to `view.json`:

```
{
  "body": {
    "sections": [{
      "items": {
        "{{#each model.items}}": {
          "type": "label",
          "text": "{{username}}"
        }
      }
    }]
  }
}
```

Instead of `"text": "{{username}}"` we will use a self-mixin.

```
{
  "body": {
    "sections": [{
      "items": {
        "{{#each model.items}}": {
          "type": "label",
          "@": "$document.adapter.label"
        }
      }
    }]
  }
}
```

Of course, this assumes that we have an `adapter.label` object at root level, so let's go back and create it in the main JSON:

```
{
  "@": "https://blah.blah/jason_skeleton.json",
  "view": {
    "@": "https://blah.blah/body.json"
  },
  "model": {
    "items": {
      "@": "dwarfs@https://blah.blah/dwarfs.json"
    }
  },
  "adapter": {
    "label": {
      "text": "{{username}}"
    }
  }
}
```

Finally we have everything we need!

---

Now we can easily turn this into a Movies app which consumes the following API:

```
{
  "response": {
    "movies": [{
      "title": "American Psycho"
    }, {
      "title": "Fight Club"
    }, {
      "title": "Matrix"
    }, {
      "title": "The Shining"
    }]
  }
}
```

To do this we:

1. Switch out `model.items`.
2. Update the `adapter.label` to `"text": "{{title}}"`.

Here's the result:


```
{
  "@": "https://blah.blah/jason_skeleton.json",
  "view": {
    "@": "https://blah.blah/body.json"
  },
  "model": {
    "items": {
      "@": "response.movies@https://blah.blah/movies.json"
    }
  },
  "adapter": {
    "label": {
      "text": "{{title}}"
    }
  }
}
```

---

# Finale

## Apps as toys
During the span of time it took me writing this single article, I could have literally made 100 apps with Jasonette. Jasonette makes it THAT easy to make apps.

With mix-ins we go further. Soon you should be able to **make apps like putting together toy parts.**

<br>

## Share your mixins!

I have created a github repository where we can share mixins.

Feel free to send pull requests to share your mixins. Also be sure to subscribe to the newsletter to stay up to date!


<form action="//textethan.us8.list-manage.com/subscribe/post?u=54e23b3fe61843c19c384fc05&amp;id=76d5746d7b" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate="novalidate">
  <input type="email" value="" name="EMAIL" class="required email" id="mce-EMAIL" placeholder="enter email" aria-required="true">
  <button type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button">Subscribe</button>
  <div id="mce-responses" class="clear">
      <div class="response" id="mce-error-response" style="display:none"></div>
      <div class="response" id="mce-success-response" style="display:none"></div>
  </div>
  <div style="position: absolute; left: -5000px;" aria-hidden="true">
      <input type="text" name="b_54e23b3fe61843c19c384fc05_76d5746d7b" tabindex="-1" value="">
  </div>
</form>

<br>

<img src='/assets/voltron.png' class='bordered'>
