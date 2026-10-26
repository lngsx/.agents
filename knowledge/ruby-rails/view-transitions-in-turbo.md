# View Transitions in Hotwire Turbo — integration recipe

This is the actual knowledge of using View Transitions inside Turbo (frames
and Drive) — quite a hack, and it was never officially documented anywhere.

Source: https://dev.to/nejremeslnici/how-to-use-view-transitions-in-hotwire-turbo-1kdi
Author: Matouš Borák (NejŘemeslníci). Posted Feb 16 2023, edited Mar 10 2023.
Banked from the article below; site noise stripped. The frame/Drive setup in
the article body was tried on a real project and works as of Oct 2026. The
Turbo-cache workaround found in the comments was never tested here. The
original article wrote its view snippets in Slim (plus Tailwind classes); the
owner converted those two snippets to ERB when banking. Everything else —
article text, JS, CSS, controller, routes, comments — is verbatim.

---

# How to use View Transitions in Hotwire Turbo

Have you heard the news? Google Chrome 111 will be released on March 1st 2023 and – among other things – will ship a feature I was eager to try since I first heard about it: View Transitions. This is a new API proposed by Google at W3C, currently as a 1st Working Draft. An informal summary for it can be found in this Explainer which seems to be a more up-to-date version of the original introductory article published at the Chrome Developers site last year. I highly recommend reading either of the documents. Update: there is now some MDN documentation available, too!

So what are View Transitions good for? In short, they allow adding animated page transitions. Although we already have several standard options to animate stuff on web pages (CSS Transitions, CSS Animations or the Web Animations API) and countless more options in particular JavaScript frameworks and libraries (Framer Motion for React, Vue Transitions, Svelte Transitions, Swup, Barba.js or Animate.css to name just a few), the web still lacks a generic, standards-based and easy-to-use solution to animate transitions between pages or during DOM updates. At least that’s what Google engineers say and I tend to agree with them.

In the Hotwire Turbo world specifically, several discussions about integrating transition animations also took place and a few promising approaches emerged, namely the Turn project or the transitions in Bridgetown. There is also a chapter in the Noel Rappin’s Modern Front-End book and an interesting article but overall, frankly, this topic still fells somewhat early-stage and exploratory.

And here comes the View Transitions proposal. Could it change the situation for Hotwire and others? It of course depends on many things, most importantly on the adoption rate by other browser vendors and the community in general and whether the W3C Draft Recommendation track succeeds. But I have a gut feeling that yes, it might change things!

## A ”Hello world“ example

Let’s take a look at a simple example. Coincidentally, a bunch of examples for an animated counter have recently been created by Jake Archibald in React, Svelte and Lit, so let’s try building the same in Turbo! I will demonstrate it in a Ruby on Rails app as this is what I’m most accustomed to but it should work the same in any other Turbo-enabled framework.

So, we will create a simple page with a big bold number and a link that will increment this number using a Turbo Frame. First, we’ll build a bare-bones page and then we’ll add a View Transition animation.

Turbo Frame needs to route it’s requests somewhere so let’s add a route first:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  resources :counter, only: :index
end
```

The corresponding controller (by the way, a ”singular“ one) can stay basically empty:

```ruby
# app/controllers/counter_controller.rb
class CounterController < ApplicationController
  def index; end
end
```

The template renders the `<turbo-frame>` tag and inside it the link and the counter itself (Tailwind styling is used, hopefully the notation is sufficiently self-explaining):

```erb
<%# app/views/counter/index.html.erb %>
<%= turbo_frame_tag :incrementing_counter do %>
  <% counter = params[:counter].to_i %>
  <%= link_to "Increment", counter_index_path(counter: counter + 1),
              class: "block p-2 bg-gray-100 active:bg-gray-200" %>

  <div id="counter" class="absolute w-full text-8xl font-bold text-center">
    <%= counter %>
  </div>
<% end %>
```

And this is how it looks so far: Turbo makes requests and replaces the Frame upon clicking the link and no animations are triggered (as can be seen in the lower-right panel):

Now, let’s add View Transitions to the Turbo Frame. To do this, we need to override the default rendering function for Turbo Frames in the `turbo:before-frame-render` event handler with a custom one that utilizes View Transitions:

```js
// app/javascript/application.js
addEventListener("turbo:before-frame-render", (event) => {
  if (document.startViewTransition) {
    const originalRender = event.detail.render
    event.detail.render = (currentElement, newElement) => {
      document.startViewTransition(()=> originalRender(currentElement, newElement))
    }
  }
})
```

The handler code first checks whether View Transitions are supported by the browser and if so, it wraps the original rendering function with the `document.startViewTransition` function. And that’s it! These couple of lines trigger a default cross-fade animation of the whole page whenever any Turbo Frame renders. Yes, the whole page is indeed animated by default but since most elements on the page are either identical (those outside the Turbo Frame) or replaced by identically looking elements (e.g. the link inside the Frame), in practice only the counter itself feels animated. It looks like this:

Finally, we can customize the transition a bit. To match the examples mentioned above, let’s rotate the counter during transitions. For this we will need to utilize ”named“ View Transitions, explained here in the documentation. Whenever we select a part of the screen with a CSS selector and name it using the `view-transition-name` attribute, the browser starts to animate the corresponding screen region independently of all other parts. This way, we can animate one or more elements on the page without affecting the rest of the page.

So let’s add the following CSS to the index template (this just renders a normal `<style>` tag):

```erb
<%# app/views/counter/index.html.erb %>
<%# (anywhere outside the Turbo Frame tag) %>
<style>
  /* (1) */
  #counter {
    view-transition-name: counter;
    contain: layout;
  }

  /* (2) */
  @keyframes rotate-out {
    to {
      transform: rotate(90deg);
    }
  }

  @keyframes rotate-in {
    from {
      transform: rotate(-90deg);
    }
  }

  /* (3) */
  ::view-transition-old(counter) {
    animation-duration: 200ms;
    animation-name: -ua-view-transition-fade-out, rotate-out;
  }
  ::view-transition-new(counter) {
    animation-duration: 200ms;
    animation-name: -ua-view-transition-fade-in, rotate-in;
  }
```

Let’s break this code down a bit:

- (1) The CSS selector `#counter` matches the counter div and the `view-transition-name` property names this area of the screen, for the purpose of View Transitions, as `counter`. This name will be used in the animation declarations below.
- (2) The clone property currently must be added here for some reasons internal to the current View Transitions implementation in Chrome and must be set to `paint` or `layout`. This restriction is planned to be removed from the specification, though, and in fact I’ve heard that it is not needed in Chrome Canary any more.
- (3) The rotation animation keyframes are defined here. Note that while the transition also uses fade-in and fade-out animations, they don’t have to be defined here because the spec requires browsers to implement them natively under the name `-ua-view-transition-fade-in/out`.
- (4) The CSS animations for the counter (the View Transition area named `counter`) are configured here. The CSS selectors here are some of the pseudo-elements automatically created during the transition. The `-old` pseudo-element represents a screenshot of the old DOM state that should somehow disappear or ”go away“ from the viewport and the `-new` pseudo-element represents a live version of the final DOM state that should be brought into sight.

So, overall, this code selects a portion of the page and animates it independently from the rest of the page during Turbo Frames DOM updates. Behind the scenes, the default cross-fade for the rest of the page still also takes place, it just is not visible because all its elements are visually identical. The result looks like this:

## A few initial tips & tricks

### Does this work for Turbo Drive visits, too?

Sure it does and it’s actually pretty easy! All we have to do is define the same event handler as we did above but attach it to the `turbo:before-render` event instead. By default we’ll get a cross-fade animation of the whole page during Turbo Drive page visits.

### Do not try to ”name“ the Turbo Frame itself

When playing with Turbo Frame View Transitions I first tried to use a custom animation for the whole Turbo Frame element by naming it via the `view-transition-name` property. For some reason, this does not work and you end up with a very cryptic and misleading error message in the console (yes I did have the contain property in the CSS declaration):

```
Aborting transition. Element must contain paint or layout for view-transition-name : counter
```

So, when using custom animations, an element from inside the Frame must be selected and named.

### Debugging View Transitions

Since View Transitions are technically just normal CSS animations, they can be inspected with the Animations panel in the Dev Tools. Also, the automatically created pseudo-elements are visible in the Elements tab during the transitions:

## Conclusions

I confess I am quite excited about the new View Transitions API. Among the things I particularly like about it are the following:

- It is surprisingly easy to plug this inside Hotwire Turbo and you get the default cross-fade transition animation immediately for free (in latest Chrome-like browsers, that is).
- Since this is implemented natively in the browser, the animations are highly optimized and performant.
- View Transitions should allow (today or in the future) building highly interactive transitions similar to those in Material Design.
- There is some initial support for Multi-Page Applications, too, which is great news because we can bring transition animations declared in CSS to our old but gold apps.
- It should be possible to use a different animation based on the ”direction“ of the visit (Back/Forward) using the Navigation API (also still experimental and not very well supported, though).

Things I am still concerned about:

- Browser support: the Firefox team evaluates it, the Safari team is silent. This will be a log run and making a polyfill is probably too difficult. For web sites where transition animations are critical, this is still a no go.
- If you’re not careful enough, the transition feels more fluid but also a little bit slower. The reason for it is that View Transitions start the animations at the moment when both the old and new DOM states are already rendered. This means that the exit animation is delayed until new content is available and until that time, nothing happens. Also, the entry animations for the new state usually delay its appearance a little bit more.
- This is not a problem of View Transitions themselves but rather a more generic one. If the exit animation (e.g. a fade out) started immediately after user interaction (e.g. a link click), sometimes the user would have to stare at a blank page until the new page content is grabbed, rendered and run through an entry animation. Still, some kind of support for this scenario (possibly with custom loaders or skeletons) would be nice.
- Tailwind support: I think the current Tailwind syntax does not allow targeting the HTML document-connected pseudo-elements so we have to resort to custom CSS (which is not a big problem, actually).
- All transitions target the whole page, there is currently no option to make, say, two components (Frames) animate totally independently. An initial proposal for ”scoped transitions“ can be found here.

Overall, I like this feature and wish it matures enough and gets wider support soon!

---

## Comments (findings from the thread)

```
tajakobsen (Tom Arild Jakobsen) • Mar 15 '23

I think this is really cool, so I tried to experiment with it a little.

I wanted to do some transitions on turbo:before-render, but I'm having a problem with my Stimulus-controllers being applied again when I navigate back.

I also get the exact same behavior (both positive and negative) with this code:

addEventListener("turbo:before-render", (event: TurboBeforeRenderEvent) => {
  if (document.startViewTransition) {
    event.preventDefault();

    document.startViewTransition(() => {
      event.detail.resume();
    });

  }
});

Any thoughts?
```

```
borama (Matouš Borák) • Mar 16 '23

Hi Tom, what do you mean by "being applied again"? Can you please elaborate?
```

```
tajakobsen (Tom Arild Jakobsen) • Mar 16 '23

Hi Matouš. Sure, I'll give an example.

On my header menu I have some buttons that are created by a Stimulus-controller (because I want to know that JavaScript exists before I add them to the DOM.)

Every time I navigate back another set of those buttons are added to the header menu (for that page).
```

```
borama (Matouš Borák) • Mar 17 '23

I must be still missing something as I think this is just the default behavior of Stimulus…? When you add the buttons in a connect() method, that method gets called every time you visit the page (regardless whether by link or the back button), doesn't it? Are you saying that the code behaves differently with and without the View transitions before-render listener attached?
```

```
tajakobsen (Tom Arild Jakobsen) • Mar 17 '23 • Edited on Mar 17

Yes, it behaves differently. It is related to the cache that turbo keeps of visited pages.

The problem is that when a page is loaded from that cache (+ viewTransition is used), the JavaScript is applied a second time to that same DOM (including Stimulus).

Adding <meta name="turbo-cache-control" content="no-cache"> to the page will fix the issue. So I will keep my view-transitions and go with that solution for now.

I will also raise this issue on the forum, and see if anyone else has seen this.
```

```
borama (Matouš Borák) • Mar 17 '23

Aah, thanks, I see, the problem is that we run our web with Turbo caching disabled (which I forgot about) so I never encountered this issue in the first place. I'll play with it some more one day…
```

```
bkspurgeon (Ben Koshy) • Jun 15 '23

Thanks for the post.

It looks like it's in very early stages. The custom CSS is painful - perhaps we need a PostCSS tool or similar tool to simplify? Ideally I would just like to add some type of data attribute on a html element, and let hotwire / tailwind take care of the rest - hopefully the PR gods devise something to make this easier in future.
```

```
borama (Matouš Borák) • Jun 19 '23

I agree, the CSS part seems the most cumbersome. Recently, I saw a demo from Hey so perhaps they will add some kind of support to Hotwire soon…
```
