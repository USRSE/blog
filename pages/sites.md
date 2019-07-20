---
layout: page
permalink: /sites/
---

## Community Sites

The following Research Software Communities maintain individual sites that include
biographies, and articles written by research software engineers. Browse the groups
below, or use the [Find an RSE](https://us-rse.org) to search across them.

<table class="table table-responsive">
  <thead>
    <tr>
      <th style="text-align: left">Community</th>
      <th style="text-align: left">Site</th>
    </tr>
  </thead>
  <tbody id="table-body"> <!-- sites will be appended here using GitHub Api-->
  </tbody>
</table>

<script src="{{ site.baseurl }}/assets/vendor/jquery/jquery.min.js" ></script>
<script>
(function() {
window.data = {}

$.getJSON("https://api.github.com/orgs/{{ site.github_username }}/repos", {
  format: "json"
}).done(function(data) {
  console.log("Found repos starting with {{ site.prefix }}:")
  $.each(data, function(key, value) {
    if (value.name.startsWith("{{ site.prefix }}")) {
      if (value.name != "community-template") {
       var name = value.name.replace("{{ site.prefix }}", "").toUpperCase();
       var url = "https://us-rse.org/" + value.name
       var row = "<tr style='text-align: left'><td><a href='"+ url + "'>" + name + "</a></td><td>" + value.description + "</td></tr>";
       $("#table-body").append(row);
       }
     }
  });
});
})();

</script>
