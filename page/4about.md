---
layout: page
title: About
permalink: /about/
icon: heart
type: page
comments: true
---

* content
{:toc}

## About the site
This is a personal website of DingRui, a fork from GaoHaoYang([https://github.com/Gaohaoyang/gaohaoyang.github.io](https://github.com/Gaohaoyang/gaohaoyang.github.io)).

### Admin tools
* site [configuration file](https://github.com/LiXizhi/lixizhi.github.io/blob/master/_config.yml)
* Jekyll + [lixizhi.duoshuo.com](http://lixizhi.duoshuo.com/admin/)
* Jekyll + [lixizhi.disqus.com](http://lixizhi.disqus.com/admin/)
* Jekyll theme templates: [jekyllthemes.org](http://jekyllthemes.org)
   * Theme used: [cool-concise](http://jekyllthemes.org/themes/cool-concise-high-end/)
* Jekyll official site: [jekyllrb.com](http://jekyllrb.com)
* YAML for human readable markdown: [yaml.org](http://www.yaml.org/)
* markdown reference: [kramdown](http://kramdown.gettalong.org/quickref.html)

### About comments
Add a variable called `comments` to the [YAML front matter](http://jekyllrb.com/docs/frontmatter/) and set its value to true. A sample might look like:

    ---
    layout: post
    comments: true
    # other options
    ---



### Sample markdowns
Click view source at the bottom of the page

Embedding code

* 1
{% highlight lua %}
local function main()
    print("hello world everyone")
end
{% endhighlight %}


* 2
    local function main()
        print("hello world everyone")
    end

* 3

```python
local function main()
    print("hello world everyone")
end
```


## Comments

{% include comments.html %}
