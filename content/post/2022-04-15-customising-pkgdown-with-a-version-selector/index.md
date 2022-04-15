---
title: Customising pkgdown with a version selector
author: Nic Crane
date: '2022-04-15'
slug: customising-pkgdown-with-a-version-selector
categories:
  - R
  - JavaScript
tags: []
---

One of the things I've been working on lately is implementing a version for the Arrow pkgdown site, so that users can browse to previous versions of the documentation.  In this post, I discuss the problem and the implemented solution.

## The problem

I was excited to work on this feature as its importance is clear to me from my consulting days; I've worked with many organisations who have had to lock down versions of R packages they allow employees to install, and so upgrading to the latest version isn't an option.  Arrow is growing and developing as a project, both in terms of internal functionality and nudges to improve package UX, and so the latest docs version may be inaccurate if users are working with a version of the package from 1 year ago, given we do major releases every three months.

## Does this already exist in pkgdown?

It should be noted that pkgdown does already support having documentation for dev and release versions of packages, which is sufficient for most cases, but it doesn't currently support more versions than that.  This has been discussed [in an issue](https://github.com/r-lib/pkgdown/issues/1373), and I'd considered trying to do something which I could submit as a PR to pkgdown, but it's complicated by having to make a number of design decisions which would work at a general level (and not just specific to Arrow), and, frankly, not having the resources to invest the time and effort required to come up with this kind of solution.

Instead, at the suggestion of a colleague, I opted for a simpler solution - using custom JavaScript to override the version badge on the page and replace it with a dropdown selector.

## The solution

Arrow already has the CI set up to deploy different versions of the docs to specific URLs - the task here is just to point pkgdown to the right place.  The docs are deployed to the following URLs:
* https://arrow.apache.org/docs/r/ - the released version
* https://arrow.apache.org/docs/dev/r/ - the dev version
* https://arrow.apache.org/docs/<arrow release version number>/r/ - a previous version

When creating a pkgdown site, you can [display the package version in the navbar](https://pkgdown.r-lib.org/reference/build_site.html).  Here's how this looks on an older version of the Arrow R package docs:

<img src="images/hoverversion.png" alt="screenshot of arrow pkgdown site showing badge for version 5.0.0 of the package with mouse hovering over it and tooltip showing phrase &quot;Released version&quot;" width="30%"/>

The plan was to use jQuery to grab this element from the page and replace it with a dropdown that has the current version selected but allows the user to navigate to another version.

### Versioning numbering

One of the constraints that I had to work with is that the deployed URL version number might not match up to the R package version number.  This is because the deployed URL reflects the overall Arrow project release number, but we can release revisions to the R package independently.  This might sound a bit confusing, so let's break it down with an example.  Keep this in mind - the Apache Arrow project does an overall release; this includes, amongst other things, the Arrow C++ implementation, upon which the R package is based.  The R package is subsequently sent to CRAN.  If we find any bugs in the R packages, we can (independently of the overall project's releases) submit updates of the R package to CRAN.  In the timeline below, I refer to the releases from the main project as "Apache Arrow [version]" and R package submissions to CRAN as "arrow pkg [version]".

* 2021-04-26 Apache Arrow 4.0.0 is released
* 2021-04-26 arrow pkg 4.0.0 is on CRAN
* 2021-05-10 arrow pkg 4.0.0.1 is on CRAN
* 2021-05-26 Apache Arrow 4.0.1 is released
* 2021-05-28 arrow pkg 4.0.1 is on CRAN

I've deliberately picked a release that had a higher number of updates than usual, both from the main project and on CRAN, but hopefully, you can see from this how the different version numbers correspond with one another.

Anyway, the deployed docs only go to the minor version number - so in this case, https://arrow.apache.org/docs/4.0/r/.  If we were looking at the docs on 11th April 2021, the pkgdown site would reflect the latest released version of the R package, so 4.0.0.1.  If we were looking at that same URL on 29th May 2021, the pkgdown site would then display the latest released version, which would be 4.0.1, as you can see in the example below.

<img src="images/Screenshot from 2022-04-15 11-23-43.png" alt="" width="50%"/>

The way I solved this mismatch was to use the solution already used in the main docs - maintain a JSON file mapping from the URL of the deployed docs to the R package version.  I did initially think that this was a little overcomplicated and thought about doing it all with JavaScript, but I realised that to do this, I'd literally need to load each URL, extract the R package version from the version badge at the top, and then construct the selector - which is definitely much less efficient than loading a JSON file full of mappings.  Instead, I created a JSON file containing key/value pairs; "name" - the displayed name of the package version, and "version" - the part of the URL following "https://arrow.apache.org/docs/".

Here's the latest (at the time of writing) version of that JSON file:

```
[
    {
        "name": "7.0.0.9000 (dev)",
        "version": "dev/"
    },
    {
        "name": "7.0.0 (release)",
        "version": ""
    },
    {
        "name": "6.0.1",
        "version": "6.0/"
    },
    {
        "name": "5.0.0",
        "version": "5.0/"
    },
    {
        "name": "4.0.1",
        "version": "4.0/"
    },
    {
        "name": "3.0.0",
        "version": "3.0/"
    },
    {
        "name": "2.0.0",
        "version": "2.0/"
    },
    {
        "name": "1.0.1",
        "version": "1.0/"
    }
]
```

This JSON file then just needs updating with every release - I also updated our release scripts to do that, though I won't go into detail about that here.

### Overriding the version element with a dropdown

The JavaScript code I used to create the dropdown is shown below; it's pasted from the most recent (at the time of writing) version of the code.  To summarise, it:

* sets up function `$pathStart`, the bit of the URL before `/docs/`
* sets up function `$pathEnd`, the bit of the URL after `/docs/` 
* fetches the JSON file mapping R package versions to URLs
* creates the appropriate links based both on the desired version number and the current page (i.e. so if I'm browsing the documentation for a function, I'll be redirected to the selected version's documentation for that function, rather than the main docs page for that version)
* sets up a function that, when an item is selected, quickly checks that the page selected exists and, if not, redirects the user to the main docs page for that version (i.e. if I am browsing a function's docs in the current version and navigate to a version where that function didn't exist)
* creates the selector and objects and replaces the "version" span with the dropdown instead

Just for transparency, I'll note that the idea for the function to check the page exists and redirect is based on a colleague's implementation of this in the main docs, which were based on something else, though I did have to implement it myself here to make it work with my code.  Yay for open source and not reinventing the wheel :)

```
$(document).ready(function () {

  /**
   * This replaces the package version number in the docs with a
   * dropdown where you can select the version of the docs to view.
   */

     // Get the start of the path which includes the version number or "dev"
     // where applicable and add the "/docs/" suffix
    $pathStart = function(){
	    return window.location.origin + "/docs/";
    }


    // Get the end of the path after the version number or "dev" if present
    $pathEnd  = function(){
      var current_path = window.location.pathname;
      return current_path.match("(?<=\/r).*");
    }

    // Load JSON file mapping between docs version and R package version
    $.getJSON("https://arrow.apache.org/docs/r/versions.json", function( data ) {
      // get the current page's version number:
		  var displayed_version = $('.version').text();
		  // Create a dropdown selector and add the appropriate attributes
      const sel = document.createElement("select");
      sel.name = "version-selector";
      sel.id = "version-selector";
      sel.classList.add("navbar-default");
      // When the selected value is changed, take the user to the version
      // of the page they are browsing in the selected version
      sel.onchange = check_page_exists_and_redirect;

      // For each of the items in the JSON object (name/version pairs)
		  $.each( data, function( key, val ) {
		    // Add a new option to the dropdown selector
        const opt = document.createElement("option");
        // Set the path based on the 'version' field
        opt.value = $pathStart() + val.version + "r" + $pathEnd();
        // Set the currently selected item based on the major and minor version numbers
        opt.selected = val.name.match("[0-9.]*")[0] === displayed_version;
        // Set the displayed text based on the 'name' field
        opt.text = val.name;
        // Add to the selector
        sel.append(opt);
	    });

      // Replace the HTML "version" component with the new selector
      $("span.version").replaceWith(sel);
    });
});
```

## Testing

I considered testing this locally, but it was tricky because the code for breaking up the URL didn't quite work properly when running this on localhost.  I could have tried to fix that, but I'd also have to simulate having multiple versions of deployed docs, and ultimately it was much simpler to test on the live docs instead.  

### Local overrides

Originally I tested this all by opening up the developer tools in a web browser on the live site and pasting the JS above into the console.  This was fine, though later on, when deploying some fixes to a couple of minor bugs, I learned about [local overrides](https://webkit.org/web-inspector/local-overrides/).  Basically, this is a feature of developer tools that allows you to specify JS to load when the page loads instead of whatever is deployed on the site (or perhaps as well as?) - this was invaluable when testing a fix to a bug that only appeared when browsing between pages.

The bugs found in the initial implementation are discussed below.

### Issue 1 - cached HTML

One [problem](https://issues.apache.org/jira/browse/ARROW-15895) was that for one user when they browsed to a page that contained the version selector (let's call it Page A), navigated to another page (let's call it Page B), and then hit "back", the selected/displayed version was the version from Page B and not Page A, as expected, given we were at the URL for Page A.

This felt like a caching problem to me, and whilst I couldn't replicate it locally, I had a bit of a search and found similar StackOverflow questions which indicated it was to do with this, so I added this bit of JS code which basically checks if the page is loading from a cache and if so, reloads it.

```
$(window).bind("pageshow", function(event) {
  if (event.originalEvent.persisted) {
    window.location.reload()
  }
});
```

Weirdly, this seemed to affect users on Chrome on macOS but not on Chromium on my Linux machine - perhaps the Chrome/Chromium defaults are a bit different, or we just had different versions with different things going on.  I asked a colleague who could replicate the bug on his MacOS machine to test my solution using local overrides as mentioned above, and this seems to have fixed it quite nicely.

### Issue 2 - not showing on Safari on macOS

Another colleague found that the dropdown selector wasn't showing up at all on Safari, and they could only see the original 'version' tag but no dropdown.  It took a while to work this one out, and eventually, I asked a colleague on MacOS to dump out the output from the JS console.

The problem lay in the `$pathEnd` variable, defined as:

```
$pathEnd  = function(){
  var current_path = window.location.pathname;
  return current_path.match("(?<=\/r).*");
}
```

The issue here is that part of the regular expression is using a positive look-behind - basically saying "look '/r' and return everything after it".  As someone who doesn't use regex often, I felt very smug putting together something "clever" like this, but these kinds of expressions aren't supported by all browsers, so in the end, I simplified it to a regex that finds the entire chunk, "/r" and all, and then just returns everything after the second character.  

```
$pathEnd  = function(){
  var current_path = window.location.pathname;
  // Can't do this via a positive look-behind or we lose Safari compatibility
  return current_path.match("\/r.*")[0].substr(2);
}
```

## Conclusion

Awesome, thanks for sticking with me as I walked through that.  This was honestly a bit of a PITA to do as I'm not the best JS developer, and the bugs were a pain to solve without being able to easily replicate them with my browsers/OS (I'm sure there are tools out there to do it but honestly just having a quick call with a colleague was the simplest solution!)  I would have loved to have done something where I ended up submitting a PR to pkgdown to make this a more general feature, but this solution is super-custom and isn't set up that way.  That said, this was a super interesting problem to solve, and it was good fun working out how to fit all the pieces together!


<img src="images/finalthing.png" alt="screenshot of Arrow pkgdown site with dropdown selector clicked on" width="100%"/>