# Making My Linux Notifications Look Like Frosted Glass 🧊

So my notifications were... fine. Boring, but fine. Square corners, a flat solid
background, very 2010. I wanted that soft, modern, frosted-glass vibe instead — rounded
corners and a blurry see-through panel. Turns out it's a quick change. Here's how I did it
on my [Omarchy](https://omarchy.org/) setup (that's Arch Linux + Hyprland).

Here's where I ended up:

![Frosted notification with rounded corners and a blurred background](images/notification-closeup.png)

See how the wallpaper behind it is all soft and blurry? That's the bit I was after.

## The one thing that confused me at first

I assumed the notification app would do the blur. It doesn't. The job is split between two
programs:

- **Mako** draws the notification — it handles the rounded corners and how see-through the
  background is.
- **Hyprland** (the thing that draws your whole desktop) does the actual blurring of
  whatever's *behind* the notification.

So you make the background transparent in one place, and blur it in the other. Once that
clicked, the rest was easy.

## Step 1: Find the notification config

On Omarchy, notifications are Mako. The config is at `~/.config/mako/config`. Heads up
though — on my machine that's a shortcut (symlink) pointing into my theme:

```bash
$ readlink -f ~/.config/mako/config
/home/iaakash/.config/omarchy/current/theme/mako.ini
```

So the *real* file I needed to edit was `mako.ini` in the theme folder. Easy to miss.

It started out looking like this — notice there's no rounding and the background is fully
solid:

```ini
text-color=#ffcead
border-color=#7d82d9
background-color=#060B1E
```

## Step 2: Round the corners and make it see-through

I changed it to this:

```ini
text-color=#ffcead
# kill the border
border-size=0
# round the corners
border-radius=14
# transparent background — those last 2 letters/numbers are the see-through level
background-color=#060B1E99
```

The magic is in that `99` on the end of the color. It's the transparency dial:

- `cc` ≈ pretty solid (80%)
- `99` ≈ nicely see-through (60%) ← what I used
- `66` ≈ very ghostly (40%)

Lower number = more transparent. Mess with it until it feels right.

Then I told Mako to reload so it'd pick up the changes:

```bash
makoctl reload
```

At this point the background was transparent, but it just showed the plain desktop through
it — no blur yet. Kind of looked like a glitch, honestly. That's the next step.

## Step 3: Add the blur

The blur lives in Hyprland. You tell it "blur this specific thing," and the "thing" has a
name. To find the name, I popped a test notification and asked Hyprland what was on screen:

```bash
notify-send "test" "hi"
hyprctl layers | grep namespace
# ... namespace: notifications  ← there it is
```

It's called `notifications`. So I opened `~/.config/hypr/looknfeel.conf` and added one line:

```ini
# blur whatever's behind the notifications so they look frosted
layerrule = blur on, match:namespace notifications
```

Reload, and check nothing's broken:

```bash
hyprctl reload
hyprctl configerrors   # silence = good
```

And boom — frosted glass. 🎉

## Two little traps I fell into

If you're on a recent Hyprland (I'm on 0.55.2), watch out:

1. It's `blur on`, **not** just `blur`. The old short version threw an error at me
   (`invalid field blur: missing a value`). They made you spell out the `on` now.
2. There used to be a companion line called `ignorezero` — it's gone in this version and
   throws an error if you add it. You don't need it here anyway, so just skip it.

If you copy-paste old guides from the internet, these two will trip you up.

## A wider shot for context

Here's one with more of the desktop showing, so you can see the blur blending into the
wallpaper:

![Notification in the corner of the desktop](images/notification-context.png)

## Want to tweak it yourself?

| If you want... | Change this | Where |
|----------------|-------------|-------|
| More/less rounded | `border-radius=14` | `mako.ini` |
| More/less see-through | the `99` in `#060B1E99` | `mako.ini` |
| The border back | `border-size=2` (+ pick a color) | `mako.ini` |
| Heavier blur | the `passes` number in Hyprland's `blur { }` block | `looknfeel.conf` |

(Heads up: that blur `passes` setting is global, so cranking it affects everything that
blurs, not just notifications.)

## That's it

Two files, a handful of lines:

- `~/.config/omarchy/current/theme/mako.ini` — corners + transparency
- `~/.config/hypr/looknfeel.conf` — the blur line

Took maybe five minutes and my notifications look ten times nicer. Try it out with:

```bash
notify-send "Hello" "Looking good 😎"
```

*Setup: Omarchy / Arch Linux / Hyprland 0.55.2 / Mako.*
