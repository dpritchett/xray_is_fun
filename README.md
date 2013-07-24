# Crack open a nested template with xray-rails

Hi, I'm [Daniel Pritchett](http://dpritchett.net).  I work here at [Coroutine](http://coroutine.com).  We help our clients grow their businesses by delivering :satellite: mission-critical software that works.

This presentation is written up for the[@MemphisRuby](http://twitter.com/memphisruby) Users Group meeting in July 2013.

#### Want to follow along online? Point your tricorder below:

![QR link to this repo](http://bitly.com/13bjKyP.qrcode)

### What's all this?
[Xray-rails](https://github.com/brentd/xray-rails) provides an in-browser overlay to allow developers to identify which template generated each section of the rendered page. ([Animation](http://f.cl.ly/items/1A0o3y1y3Q13103V3F1l/xray-rails-large.gif))

### What's that mean?
Rails templates allow the mixing of dynamic content with static content.

```erb
Hello, my name is <%= user.name.upcase %>.
```

Becomes:
```
Hello, my name is DANIEL PRITCHETT.
```


Repeated template elements can be broken out into partial templates.  Define a footer partial template:

```erb
# app/views/shared/_footer.html.erb
<div id="footer>
  Copyright <%= DateTime.now.year %> Memphis Ruby Users Group
</div>
```

Include it in a top-level template:

```erb
# app/views/demo/some_random_page.html.erb
This is an example page!
<%= render partial: 'shared/footer' %>
```

Combined output:

```
This is an example page!

Copyright 2013 Memphis Ruby Users Group
```

### Okay, but why do I need xray?
Larger web applications can have tens, hundreds, maybe even thousands of template files.  When you're looking to update a screen in an old app that someone else put together it really helps to have tools that help you home in on the code responsible for the on-page DOM elements that need changing.  'Myself eighteen months ago' _totally_ counts as someone else!

```
> find app/views | wc -l
     115
```

### Fine.  How do I add it to my Rails app?
```ruby
# From your app's Gemfile:
group :development, :test do
  # ...
  gem 'xray-rails', '~> 0.1.6'
end
```

### How does it work?
```
~/.rvm/gems/ruby-2.0.0-p195/gems/xray-rails-0.1.6/lib $ ack chain
xray/engine.rb
37:        alias_method_chain :render, :xray
```

#### What's alias_method_chain again?

Check out this sweet phone dialer utility:
```ruby
class ReallySecureDialer
  def place_call(to_number)
    # ... do stuff
  end
end
```

Now enhance it!
```ruby
require 'prism'

ReallySecureDialer.class_eval do
  def place_call_with_log
    # capture results of the original :place_call method
    call = self.place_call_without_log
    
    # store metadata in secure location
    Prism.phone_home call.metadata
  end

  alias_method_chain :place_call, :log
end
```

With this in place, every invocation of `.place_call` is automatically :notebook: logged.

## Ok, so how does xray's server-side code injection shake out in the browser?

<del>Java</del>CoffeeScript listener standing by to show the xray overlay:
```coffee
Xray.init = do ->
# ...
  # Register keyboard shortcuts
  $(document).keydown (e) ->
    # cmd+shift+x on Mac, ctrl+shift+x on other platforms
    if (is_mac and e.metaKey or !is_mac and e.ctrlKey) and e.shiftKey and e.keyCode is 88
      if Xray.isShowing then Xray.hide() else Xray.show()
    if Xray.isShowing and e.keyCode is 27 # esc
      Xray.hide()

# ...

# Scans the document for templates, creating Xray.TemplateSpecimens for them.
Xray.findTemplates = -> util.bm 'findTemplates', ->
  # Find all <!-- XRAY START ... --> comments
  comments = $('*:not(iframe,script)').contents().filter ->
    this.nodeType == 8 and this.data[0..9] == "XRAY START"
```

Where'd `<!-- XRAY START -->` come from?  Our `template.render` alias chain!
```ruby
  def self.augment_template(source, path)
    id = next_id
    augmented = "<!--XRAY START #{id} #{path}-->\n#{source}\n<!--XRAY END #{id}-->"
    ActiveSupport::SafeBuffer === source ? ActiveSupport::SafeBuffer.new(augmented) : augmented
  end
```

## Ok, we know how xray works.  Now what can we do with it again?
Play!

Demo code is published: https://github.com/dpritchett/xray_is_fun
