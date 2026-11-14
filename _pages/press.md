---
layout: page
title: press
permalink: /press/
nav: true
nav_order: 4.5
# Shown as a "featured in" line at the top of the page.
featured_in: [Reuters, BBC, Wired, Forbes, Scientific American, Smithsonian, NBC News, The Telegraph, IEEE Spectrum, Ars Technica, STAT]
---

<!-- Stories live in _data/press.yml; markup below, styles in _sass/_press.scss. -->

<p class="press-featured">
  <span class="press-featured-label">Featured in</span>
  {{ page.featured_in | join: ' <span class="press-dot">·</span> ' }}
</p>

<div class="press-grid">
  {% for story in site.data.press %}
    {% assign quote_link = story.coverage | where: 'outlet', story.quote_by | first %}
    <div class="press-card">
      <img class="press-thumb" src="{{ story.image | prepend: 'assets/img/' | relative_url }}" alt="{{ story.title | escape }}" loading="lazy">
      <div class="press-body">
        <div class="press-title">{{ story.title }} <span class="press-year">{{ story.year }}</span></div>
        {% if story.quote %}
          <p class="press-quote">
            {% if quote_link %}<a href="{{ quote_link.url }}" target="_blank" rel="noopener noreferrer">“{{ story.quote }}”</a>{% else %}“{{ story.quote }}”{% endif %}
            {% if story.quote_by %}<span class="press-quote-by">— {{ story.quote_by }}</span>{% endif %}
          </p>
        {% endif %}
        <div class="press-outlets">
          {% for c in story.coverage %}
            <a
              class="press-outlet"
              href="{{ c.url }}"
              target="_blank"
              rel="noopener noreferrer"
              {% if c.archived %}title="Archived copy (original link no longer works)"{% endif %}
            >{{ c.outlet }}</a>
          {% endfor %}
        </div>
        {% if story.paper %}
          <a class="press-paper" href="{{ story.paper.url }}" target="_blank" rel="noopener noreferrer">{{ story.paper.name }} paper →</a>
        {% endif %}
      </div>
    </div>
  {% endfor %}
</div>
