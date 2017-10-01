---
title: Reporting mal anders, Markdown sei Dank
subtitle: Folien für DOAG Konferenz 2016, Nürnberg
lang: de
data-separator-notes: "^Anmerkung:"
---
<script>
window.to_png = function(){
  var images = document.getElementsByTagName('img');
  for (var i = 0; i < images.length; i++) {
    var image = images.item(i);
    var source = image.getAttribute('src');
    if (source.indexOf(".png.svg") !== -1) {
      image.setAttribute('src', source.replace(".png.svg",".svg.png"));
    }
  }
}
window.to_svg = function(){
  var images = document.getElementsByTagName('img');
  for (var i = 0; i < images.length; i++) {
    var image = images.item(i);
    var source = image.getAttribute('src');
    if (source.indexOf(".svg.png") !== -1) {
      image.setAttribute('src', source.replace(".svg.png",".png.svg"));
    }
  }
}
</script>
