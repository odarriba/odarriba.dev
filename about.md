---
title: About me
permalink: /about/
layout: page
excerpt: Hello peeps, I'm student of computer science from Banyuwangi, living in Jogjakarta. This blog for documentation about my programming journey, running on jekyll, hosting on netlify and using my own simple theme.
comments: false
---

Hi there! I'm a software engineer with a **strong focus on Elixir**, a language that has powered my work across various industries — from **consultancy** and **e-commerce** to **healthcare** and **travel**. My career has given me the opportunity to dive deep into different domains, solving unique challenges and building robust, scalable systems.

In addition to my professional work, I’ve created [ElixirJobs](https://www.elixirjobs.net), a platform dedicated to helping Elixir developers find opportunities in the industry. I also co-authored [ErrorTracker](https://github.com/elixir-error-tracker/error-tracker), a tool designed to make error monitoring easier for Elixir developers. These projects reflect my commitment to the Elixir community and my desire to contribute meaningfully to the tools and resources we all rely on.

My passion for technology and software engineering is what drives me every day. I love crafting **solutions that are efficient but also maintainable**. Alongside my work in software development, I have a keen interest in system administration, enjoying the complexities and rewards that come with maintaining and optimizing infrastructure.

Currently, **I work remotely** from Europe for [Bizneo](https://www.bizneo.com), an HR software company where I get to apply my skills to help shape the future of work.

When I'm not coding, you'll find me spending time with my wife María, our cat Ginebra, and our greyhound Trufa.

Thanks for stopping by my page! If you’d like to connect or learn more about my work, feel free to reach out.

<ul>
  {%- for link in site.author.links -%}
  <li><a href="{{ link.url }}" title="{{ link.name }}" target="_blank"><i class="{{ link.icon }}"></i>&nbsp;&nbsp;{{ link.name }}</a></li>
  {%- endfor -%}
</ul>
