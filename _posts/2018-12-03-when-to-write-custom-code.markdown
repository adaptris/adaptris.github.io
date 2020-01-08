---
title:             "When to write custom code"
description:       "The Interlok framework is easily pluggable the question is when?"
date:              2018-12-03 06:00:00
categories:        [interlok]
tags:              [interlok]
billboard: /billboards/icon-iot.png
author:            mcwarman
---

There are plenty of services provided out of the box, and even more within optional components. Every now and again I stumble across a problem that "requires" me to write code.<!-- more -->

Personally I try to keep it as a last resort, for supportability reasons as well as the fact it's unnecessary most of the time.

Firstly we should make a clear distinction between custom code and a new feature. A feature is piece of functionality that fills a gap or makes life easier for an Integration Engineer, which has reuse outside of the bubble you're in.

Custom code has exactly the same meaning for the first part, but then differs in the fact there isn't reuse outside of the bubble.

It's a very small difference and often one that can be argued either way. In order to try and illustrate the point we'll walk through an example (Even the example could be argued either way).


# Worked Example

The problem that we're trying to solve is we have a CSV data source that we'd like to display on a custom widget on the Interlok widgets page. In order to do that we'll expose a HTTP endpoint that the widget can consume to get its data.

Our data source looks like this:

<figure class="highlight">
{% highlight csv %}
header1,header2,header3
1,2,3
3,4,5
{% endhighlight %}
</figure>

And the widgets requires a certain format which looks like this:

<figure class="highlight">
{% highlight json %}
{
  "type" : "table",
  "data" : {
    "headers" : [ "header1", "header2", "header3" ],
    "values" : [ [ "1", "2", "3" ], [ "4", "5", "6" ] ]
  }
}
{% endhighlight %}
</figure>

Now this could be achieved by doing something like:

csv-to-json -> json-transformation

Which conceptually seems fine, until you start to try and write the JSON transformation. I haven't tried to write it but equally don't want to for reasons which anyone who's written a jolt will understand.

It's equally possible:

csv-to-xml -> xml-transformation -> xml-to-json

Which I'm not against if it plays into your skill set. But it does have a bad code stink.

Instead I took a different approach, which was to write some code:

<figure class="highlight">
  <figcaption class="g code-caption">CsvToWidgetJsonService.java</figcaption>
{% highlight java %}
@Override
public void doService(AdaptrisMessage msg) throws ServiceException {
  log.trace("Beginning doService in {}", LoggingHelper.friendlyName(this));
  ObjectMapper mapper = new ObjectMapper();
  try (Reader reader = msg.getReader();
       CsvListReader csvReader = new CsvListReader(reader, getPreferenceBuilder().build());
       Writer w = msg.getWriter();
       JsonGenerator generator = mapper.getFactory().createGenerator(w).useDefaultPrettyPrinter()) {
    String[] hdrs = csvReader.getHeader(true);
    generator.writeStartObject();
    generator.writeStringField("type", getType());
    generator.writeObjectFieldStart("data");
    generator.writeObjectField("headers", hdrs);
    generator.writeFieldName("values");
    generator.writeStartArray();
    for (List<String> row; (row = csvReader.read()) != null;) {
      generator.writeObject(row);
    }
    generator.writeEndArray();
    generator.writeEndObject();
    generator.writeEndObject();
  } catch (Exception e) {
    throw ExceptionHelper.wrapServiceException(e);
  }
}
{% endhighlight %}
</figure>

# Conclusion

What's correct is largely subjective but for me this was an example where code was the better option over the use of out of the box services.

Funnily enough I can see this code ending up being its own optional component, but as I said before there's a fine line between what makes in as a feature.

