revealjs: https://revealjs.com/ and cloned from https://github.com/hakimel/reveal.js.git.

To get this running ( credit https://gist.github.com/jsoma/629b9564af5b1e7fa62d0a3a0a47c296 )

with pandoc installed:
pandoc -t revealjs -s -o index.html slides.md -V revealjs-url=https://unpkg.com/reveal.js/ -V theme=beige
npm start

slides should be running on localhost:8000