# The Easing Blueprint: Actionable Rules for Animation

> Source: [The Easing Blueprint - animations.dev](https://animations.dev/learn/animation-theory/the-easing-blueprint)

## Core Principle

**Easing is the most important part of any animation.** It describes the rate at which something changes over time and can make a bad animation look great, and a great animation look bad.

**Key Insight:** The perception of speed is often more important than actual performance. Proper easing can make interfaces feel faster and more responsive.

---

## Easing Rules

### 1. `ease-out` ⭐ **USE MOST OFTEN**

**When to use:**

- User-initiated interactions (dropdowns, modals, buttons)
- Enter and exit animations
- Elements that appear or disappear from the screen
- Marketing page intro animations

**Why:** The acceleration at the beginning gives users a feeling of responsiveness.

**Pro tip:** Add a subtle scale down effect (`scale: 0.97`) on button `:active` pseudo-class with a `150ms` transition to make UI feel even more responsive.

**Example use cases:**

- Dropdown menus
- Modal dialogs
- Toast notifications (though `ease` can be more elegant)
- Page intro animations

---

### 2. `ease-in-out`

**When to use:**

- Elements that are already on screen and need to move to a new position
- Elements that morph into a new shape
- Animations that stay visible throughout the transition

**Why:** Mimics natural acceleration and deceleration (like a car), making the movement feel organic.

**Example use cases:**

- Timeline components
- Page morphing animations
- Dynamic Island-style components
- Elements transitioning between states while remaining visible

---

### 3. `ease-in` ⚠️ **AVOID**

**Rule:** Generally avoid `ease-in` for UI animations.

**Why:**

- The slow start makes interfaces feel sluggish and less responsive
- Accelerates at the end, which is opposite of what we want
- Feels unnatural because our brain expects things to settle at the end of movement

**Exception:** None for UI animations. Use `ease-out` instead.

---

### 4. `linear`

**When to use:**

- Constant animations (marquees)
- Visualizing passage of time (progress bars, "hold to delete" interactions)
- 3D coin rotations

**Why:** Time passes linearly, so linear easing accurately represents time-based progress.

**Rule:** Avoid `linear` for most UI animations as it feels robotic and unnatural.

**Example use cases:**

- Marquee text scrolling
- Hold-to-delete progress indicators
- 3D coin rotation animations
- Any animation where time progression needs to be visualized

---

### 5. `ease`

**When to use:**

- Hover effects (color, background-color, opacity transitions)
- Subtle, gentle animations
- When elegance is more important than responsiveness

**Why:** Asymmetrical curve that starts faster and ends slower than `ease-in-out`, creating an elegant feel.

**Example use cases:**

- Button hover states
- Color transitions
- Background color changes
- Toast notifications (when elegance > responsiveness)

---

## Custom Easing Curves

**Rule:** Built-in easing curves often have weak acceleration. Use custom easing curves for stronger, more energetic animations.

**Resources:**

- Use custom easing curves from the blueprint (sorted from weakest to strongest acceleration)
- Custom curves make animations feel more energetic and polished

**How to create custom curves:**

- Use `cubic-bezier()` function in CSS
- Use tools like Cubic Bezier Curve Generator
- Study existing curves (e.g., iOS Sheet easing for native feel)

---

## Quick Reference Decision Tree

```
Is it a user-initiated interaction?
├─ YES → Use `ease-out`
└─ NO → Is the element staying on screen?
    ├─ YES → Use `ease-in-out`
    └─ NO → Is it a hover effect?
        ├─ YES → Use `ease`
        └─ NO → Is it visualizing time/progress?
            ├─ YES → Use `linear`
            └─ NO → Use `ease-out` (default)
```

---

## Best Practices

1. **Default choice:** When in doubt, use `ease-out` - it's the most versatile for UI animations
2. **Perception matters:** Faster-spinning spinners make apps feel faster, even with identical load times
3. **Custom curves:** Prefer custom easing curves over built-in ones for stronger acceleration
4. **Experiment:** Try different easings from the blueprint to develop intuition
5. **Reference:** Keep this blueprint handy when choosing easing types

---

## Common Mistakes to Avoid

1. ❌ Using `ease-in` for UI animations (makes interfaces feel slow)
2. ❌ Using `linear` for non-time-based animations (feels robotic)
3. ❌ Using built-in curves when custom curves would be more energetic
4. ❌ Not considering perceived performance vs actual performance

---

## Additional Resources

- [Custom Easing Curves](https://animations.dev/learn/animation-theory/the-easing-blueprint) - View the full set of custom easing curves
- [Cubic Bezier Curve Generator](https://cubic-bezier.com/) - Create and experiment with custom curves
