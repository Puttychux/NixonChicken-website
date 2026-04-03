---
name: Scroll Driven Video Animation
description: Best practices for implementing a scroll-driven video animation in a hero section.
---

# Scroll Driven Video Animation

This skill encodes the best practices for embedding a video as a scroll-driven animation within a web application, specifically targeting a hero section integration. It ensures the video plays frame-by-frame synchronized with the scroll position, respects mobile responsiveness, and does not interfere with surrounding layout elements (such as masthead, navigation, and article flows).

## Basic Implementation Strategy

1. **HTML Structure**
   Ensure the video element is given the attributes `muted`, `playsinline`, and optionally `preload="auto"` to guarantee correct decoding behavior on mobile browsers (especially iOS Safari), and to prevent the video from automatically playing or taking over the full screen.
   
   ```html
   <video id="scroll-hero-video" src="assets/video.mp4" muted playsinline preload="auto"></video>
   ```

2. **CSS Best Practices**
   The video should replace the hero image visually. It must cover its container naturally.
   
   ```css
   #scroll-hero-video {
       width: 100%;
       height: 100%;
       object-fit: cover;
       display: block;
   }
   ```
   *Note: Ensure the surrounding containers have bounded sizes (`aspect-ratio`, explicit height/width) to prevent layout shifts as the video loads.*

3. **JavaScript Scroll Sync**
   Instead of autoplaying the video, we map the scroll position to the video's `currentTime`.
   
   - **Use `requestAnimationFrame`**: Never update the DOM or video `currentTime` directly in the `scroll` event handler. The scroll event fires too frequently and can cause layout thrashing. Instead, save the scroll position in a variable and read it within a `requestAnimationFrame` loop.
   - **Calculate the Scroll Fraction**: Determine how much the user has scrolled relative to the hero section. For example, `scrollProgress = window.scrollY / (heroSection.offsetHeight)`.
   - **Determine `currentTime`**: Clamp the scroll fraction between `0` and `1`. Then set `video.currentTime = video.duration * scrollFraction`.
   - **Wait for Metadata**: Ensure `video.duration` is available before trying to scrub, by waiting for the `loadedmetadata` event.

4. **Code Example**
   ```javascript
   document.addEventListener("DOMContentLoaded", () => {
       const video = document.getElementById("scroll-hero-video");
       const section = document.querySelector(".hero-story");
       
       let scrollY = 0;
       let rafId = null;
       
       // Update scroll variable on scroll
       window.addEventListener("scroll", () => {
           scrollY = window.scrollY;
           if (!rafId) {
               rafId = requestAnimationFrame(updateVideo);
           }
       }, { passive: true });
       
       function updateVideo() {
           if (video.duration) {
               // Calculate how far we've scrolled down the section
               // Using height of section + half of viewport for a smooth fade out factor
               const maxScroll = section.offsetHeight + (window.innerHeight * 0.5);
               let progress = scrollY / maxScroll;
               
               // Clamp between 0 and 1
               progress = Math.max(0, Math.min(1, progress));
               
               // Sync video to progress
               video.currentTime = video.duration * progress;
           }
           rafId = null;
       }
       
       // Force initial frame render on load
       video.addEventListener("loadedmetadata", () => {
           updateVideo();
       });
   });
   ```

## Key Considerations
- **Do NOT auto-play**. The user's scroll depth dictates the current frame.
- **Mobile Responsiveness**: `object-fit: cover` and proper `aspect-ratio` on the wrapper guarantee it looks great on any screen size. Mobile mapping works identically because `window.scrollY` and element bounds automatically recalculate.
- **Minimal Layout Disturbance**: Wrapping the video exactly where the image would reside ensures `gap`, `padding`, and responsive grids function exactly identically without requiring massive CSS refactors.
