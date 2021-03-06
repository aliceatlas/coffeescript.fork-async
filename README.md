# CoffeeScript<sub>Async</sub>

This is a small modification to [CoffeeScript][coffeescript] which adds support for the proposed ECMAScript 7 [async function syntax][async-await]. Here we make no attempt to actually make this syntax available in JavaScript engines that don't support it natively yet (currently all of them) — in practice the idea is to chain this to [Babel][babel] or something of that sort and compile it down to its underlying representation as ECMAScript 6 promises and generators, or, via [Regenerator][babel-regenerator], further down to compatibility with ECMAScript 5 and probably most major web browsers going back at least a few years.

## Examples

From the [async/await proposal][async-await]:

<table>
<tr>
  <th>ECMAScript 6 Promises</th>
  <th>ECMAScript 7 Async/Await</th>
  <th>CoffeeScript<sub>Async</sub></th>
</tr>
<tr>
<td>
<sub><pre lang="JavaScript">
function chainAnimationsPromise(elem, animations) {
  var ret = null;
  var p = currentPromise;
  for(var anim in animations) {
    p = p.then(function(val) {
      ret = val;
      return anim(elem);
    })
  }
  return p.catch(function(e) {
    /* ignore and keep going */
  }).then(function() {
    return ret;
  });
}
</pre></sub>
</td>
<td>
<sub><pre lang="JavaScript">
async function chainAnimationsAsync(elem, animations) {
  var ret = null;
  try {
    for(var anim of animations) {
      ret = await anim(elem);
    }
  } catch(e) { /* ignore and keep going */ }
  return ret;
}
</pre></sub>
</td>
<td>
<sub><pre lang="CoffeeScript">
chainAnimationsAsync = (elem, animations) ->
  ret = null
  try
    for anim of animations
      ret = await anim elem
  catch e # ignore and keep going
  return ret
</pre></sub>
</td>
</tr>
</table>

<table>
<tr>
  <th>ECMAScript 7 Async/Await</th>
  <th>CoffeeScript<sub>Async</sub></th>
</tr>
<tr>
<td>
<sub><pre lang="JavaScript">
async function getData() {
  var items = await fetchAsync('http://example.com/users');
  return await* items.map(async(item) => {
    return {
      title: item.title, 
      img: (await fetchAsync(item.userDataUrl)).img
    }
  });
}
</pre></sub>
</td>
<td>
<sub><pre lang="CoffeeScript">
getData = ->
  items = await fetchAsync 'http://example.com/users'
  await all for item of items
    do ->
      title: item.title
      img: (await fetchAsync item.userDataUrl).img
</pre></sub>
</td>
</tr>
</table>

From [Jake Archibald](http://jakearchibald.com/2014/es7-async-functions/):

<table>
<tr>
  <th>ECMAScript 6 Promises</th>
  <th>ECMAScript 7 Async/Await</th>
  <th>CoffeeScript<sub>Async</sub></th>
</tr>
<tr>
<td>
<sub><pre lang="JavaScript">
function loadStory() {
  return getJSON('story.json').then(function(story) {
    addHtmlToPage(story.heading);

    return story.chapterURLs.map(getJSON)
      .reduce(function(chain, chapterPromise) {
        return chain.then(function() {
          return chapterPromise;
        }).then(function(chapter) {
          addHtmlToPage(chapter.html);
        });
      }, Promise.resolve());
  }).then(function() {
    addTextToPage("All done");
  }).catch(function(err) {
    addTextToPage("Argh, broken: " + err.message);
  }).then(function() {
    document.querySelector('.spinner').style.display = 'none';
  });
}
</pre></sub>
</td>
<td>
<sub><pre lang="JavaScript">
async function loadStory() {
  try {
    let story = await getJSON('story.json');
    addHtmlToPage(story.heading);
    for (let chapter of story.chapterURLs.map(getJSON)) {
      addHtmlToPage((await chapter).html);
    }
    addTextToPage("All done");
  } catch (err) {
    addTextToPage("Argh, broken: " + err.message);
  }
  document.querySelector('.spinner').style.display = 'none';
}
</pre></sub>
</td>
<td>
<sub><pre lang="CoffeeScript">
loadStory = ->
  try
    story = await getJSON 'story.json'
    addHtmlToPage story.heading
    for URL of story.chapterURLs
      addHtmlToPage (await getJSON URL).html
    addTextToPage "All done"
  catch err
    addTextToPage "Argh, broken: " + err.message
  document.querySelector('.spinner').style.display = 'none'
</pre></sub>
</td>
</tr>
</table>

[coffeescript]: https://github.com/jashkenas/coffeescript
[async-await]: https://github.com/lukehoban/ecmascript-asyncawait
[babel]: https://babeljs.io/
[babel-regenerator]: https://babeljs.io/docs/usage/transformers/other/regenerator/