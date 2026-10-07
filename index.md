---
layout: default
title: audit
---
# pages-build-audit marker 20261007

effective config as seen by Liquid (own build, no secrets):

- source: `{{ site.source }}`
- destination: `{{ site.destination }}`
- safe: `{{ site.safe }}`
- plugins_dir: `{{ site.plugins_dir }}`
- includes_dir: `{{ site.includes_dir }}` layouts_dir: `{{ site.layouts_dir }}` data_dir: `{{ site.data_dir }}` collections_dir: `{{ site.collections_dir }}`
- disable_disk_cache: `{{ site.disable_disk_cache }}` cache_dir: `{{ site.cache_dir }}`
- whitelist: `{{ site.whitelist | join: "," }}`
- plugins: `{{ site.plugins | join: "," }}`
- kramdown: `{{ site.kramdown | jsonify }}`
- github.build_revision: `{{ site.github.build_revision }}` environment: `{{ site.github.environment }}` api_url: `{{ site.github.api_url }}`
