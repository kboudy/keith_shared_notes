#elixir #phoenix #live_view

# phoenix lifecycle notes

from [this video](https://www.youtube.com/watch?v=HaK9N-kqSXU)

## what happens in mount

- http request comes in
- initial html is sent to the client
  - this guarantees a regular html page even if js is disabled
- the client renders the html
- the client sends a websocket connection to the server
- the server sends the initial state to the client
- the client renders the initial state
- ...
- note that both mount cycles (connected & disconnected) will call the mount & the handle_parms callbacks

## 3 callbacks involved in rendering liveviews

### mount

- happens first, on initial connections
- parameters
  - params - the url params
  - session
    - survives page refreshes
    - data can be stored in client cookie or on server
    - [here's a good article on them](https://pentacent.com/blog/phoenix-live-views-sessions/)
  - socket

### handle_params

- happens second
- parameters
  - params - the url params, as in `mount`
  - uri - the full url _(so you'll want to use `handle_params` is you want access to the direct url)_
  - socket
- invoked after `mount` or whenever there is a [live navigation](https://hexdocs.pm/phoenix_live_view/live-navigation.html) event
  - 3 ways of navigating in liveview
    1. full page refresh/redirect
       - `<.link href={...}>`
       - `redirect/2`
    2. inter-liveview navigation
       - `<.link navigate={...}>`
       - `push_navigate(socket, to: ~p"/pages/#{@page + 1}")`
       - this does not do a full page refresh. diffs the liveviews and re-renders
    3. patch (changing params in current liveview, without reloading)
       - `<.link patch={...}>`
       - `push_patch/2`
       - only `handle_params` is called

### render

- happens last
- happens implicitly if
  1. there is no render callback and
  2. there is a same-named heex template

## web socket debugging

- ignore the dev reload websocket
- info going from client to server is green

## common misconceptions

- you need to handle params in `handle_params`
  - if you're not using patch, you can just use `mount`
- you need to do your initial assigns in `mount`
  - you can do it in `handle_params` if you want to
  - _(you never need to do the same thing in `mount` and `handle_params`)_
    - just use `handle_params` if it might involve a patch, or requires the full url
