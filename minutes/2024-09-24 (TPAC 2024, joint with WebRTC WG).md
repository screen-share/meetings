# Joint Meeting - Screen Capture CG with WebRTC WG (2024-09-24)

Notes taken by Guido Urdaneta.

### Resources
* [Recording](https://w3c.zoom.us/rec/play/T-LmO-JA408oe-SLkiAVrdZ3JKd2ZDiV9XtUdVQKppItpqqvkBI8BrTp1SKtlcpGg3F2a0a4fukUHTvh.zRBvGwI5ycKdC-Ki?canPlayFromShare=true&from=share_recording_detail&continueMode=true&componentName=rec-play&originRequestUrl=https%3A%2F%2Fw3c.zoom.us%2Frec%2Fshare%2F7YF2uudUFn8aOBfzDmMJ_pS3EgOqAszzDSmMClQTbfllFLnH5LGIsGE5IX2bjA4.Nv40xrv1VG0AQLjG)
* [Slides](https://docs.google.com/presentation/d/1seiO4a6YJIxSKed3aQpJAbf-_j2GJWYa00btZWmRCoY/edit?usp=sharing)
* Minutes: TODO - Add self-link.

### Agenda
* Introduction
* Dynamic Surface Switching (Tove Petersson)
* Captured Surface Control (Elad Alon)
* Physical/Logical resolutions (Konrad Hofbauer)
* Element Capture (Elad Alon)

### Participants (partial list)
* In person:
   * Eric Carlson, Apple
   * Youenn Fablet, Apple
   * Elad Alon, Google
   * Erik Språng, Google
   * Eugene Zemtsov, Google
   * Fredrik Solenberg, Google
   * Guido Urdaneta, Google
   * Harald Alvestrand, Google
   * Markus Handell, Google
   * Joachim Reiersen, Meta
   * Philip Hancke, Meta
   * Erik Anderson, Microsoft
   * Dominique Hazael-Massieux, W3C
* Remotely:
   * Brad Isbell, AudioPump
   * Felipe Gomes, Dialpad
   * Konrad Hofbauer, Google
   * Tove Petersson, Google
   * Bernard Aboba, Microsoft
   * Jan-Ivar Bruaroey, Mozilla
   * Duanpei Wu, Zoom

### Introduction

Bernard introduces WebRTC WG IP policies, W3C code of conduct.
Announces related workout sessions and meetings

Elad
Announces SCCG meeting

Bernard
Announces agenda
Tips

### Dynamic Surface Switching (Tove Petersson)
Same-surface switching exists in existing UAs
Chrome allows switching between tabs
Safari between  windows/screens

Injection Model
Media from each surface is injected into a single MediaStreamTrack
Works well for applications that are agnostic to what is shared

Challenges for applications that are note:
Difficult to synchronize 
Tab,Windows, Screens are different and the application might want to treat them differently
Audio can't be added if only video was captured initially
Unclear interaction with some APIs (e.g., croptarget/RegionTarget)

Switch-Track model
Each surface produces a new track
Invariants over time:
Same type
Same operations supported
Each track provides frames from a single surface
No syncrhonization need, since tracks are separate
An audio track can be added as needed

Surface:
```
Controller.onnewsource = event => {
  video.srcObject = event.stream;  // surface tracks
} 
await getDisplayMedia();
```

Proposal
Keep injection model for same-type surface switching
Allow applications to opt-in to the switch-track model for:
Cross-type switching
Better surface for interacting with captures

Current points of discussion
Can the injection model be adapted for discussion?

Another model
Both models in parallel.
A session track for the whole capture session
Provide surface tracks for each captured surface (new surface for each new type switch)

Question: How would both model be supported in parallel
Initially you get two tracks both at the same type. One stays with injection of further switches. The other track dies with switches and replaced with a new track for the new capture
Bernard question:
Why both models?
The switch-track model is a bit more work for the app. For applications that don't care, the injection model is easier.
Using both models is an attempt to accommodate the needs of both types of applications.
Google prefers the switch-track model.

How do audio/video sync? Clear with surface tracks, less clear with injection.

Jan-Ivar: Problem is worth solving. Some concerns about surface tracks/session tracks.
A bit unclear why injection for same surface type, switching for different types.
The distinction between session and surface track seems artificial.
I would prefer switch event to be notified about source switching and have a late decision about the model.

Ran out of time

Felipe: question. New event with new stream. Why not new track on existing stream?
Answer: we don't rely on streams much anymore. Adding tracks to a stream can cause issues. Better to create a new stream or just return a new track.

Youenn: Injection works: (it's shipped) Except for the audio case, where we need to do something, all other issues are solvable.
Onnewsource event looks good. I hope we can solve the audio issues. If it's too complex revise and see if we can move from injection.

Switch-track does not work well for media recorder.

Harald: this has been discussed for a long time. Deficiencies of the injection model:
The switch track model addresses these deficiencies.
We shouldn't delay trying the new model and learn from experience and not just theoretical arguments.

Action item: Continued discussion. Reach a decision soon and start experimenting.

Chair item: Decide what to do next to move the discussion along.

### Captured Surface Control (Elad Alon)

Allow apps to scroll vertically and horizontally and zoom in/out.
 
For scrolling: sendWheel and sendZoom. 
Issue: Allowed remote control. But the WG does not want to support remote control.

Change to prevent remote control:
captureWheel (designate an element, and let the browser forward from the element to the captured tab). Approch
Restrictions on setZoomLevel:
Give a window of opportunity when a trusted event occurs on an element and only

Question: Limiting something to zoom/scroll. I want to do more than this.
postMessage.

Answer: postMessage doesn't work because captured application will not listen to events and answer messages. If you want something else (clicking, etc.) , it's out of scope for this API (there's a video portal application that might do it).

Jib: This seems to address the remote control issue.
We can bikeshed the names, but the approach looks good. Maybe there is a pattern we can derive from this that is better than transient activation.

Bernard: Put on the record that this new proposal has addressed the concerns presented.
JIB: Close the technicalw 

Bernard: People in the room think this addresses the concerns. No objections.

Youenn: What about that [...]
I would prefer a different shape. Direct forwarding, similar to captureWheel. But it's OK.

### Physical/Logical resolutions (Konrad Hofbauer)
Applications can limit gDM resolution with constraints (e.g., to conserve resources, adapt resolution/quality/bandwidth) tradeoffs.

Resolution of the surface is relevant, both physical and logical.

Consider an OS that doesn't do Retina/HiDPI adjustments, moving from a small screen to a bigger one, things become smaller.
OSes nowadays have resolution independence.
Logical resolution can remain the same size when moving to higher-res screen (maybe sharper).

Resolutions involved in capturing:
Physical width/height
Logical width/height 
 Track constraints (indication of what the app wants)
Track actual width/height (Settings). What you actually get.

Today we have width/height constraints and settings.

We would like to have physical and logical surface height.

devicePixelRatio and window.screen are about the  browser tab, not captured content.

Proposal:
Expose physical and logical width/height.

Usage examples:
Capture/send at logical resolution
2 spatial quality layers: physical + logical resolution
Choose resolution based on content (text/sheets in physical, videos in logical)
Up-render small capture surfaces (uprender a 300x200 slide deck to 1080p)


Proposed API Changes

capturedSurfacePhysicalWidth/Height properties in tracks
capturedSurfaceLoficalWidth/Height properties in tracks

Question: Do we agree?

JIB: This is a good use case. I agree with the properties. Disagree to put them on the track. Put them on the controller. Would that work? Answer: Yes. 
JIB: Would work better on a capture controller.

Youenn: I like it in settings. It's easy to implement. There is an event onconfigurationchange that supports changes. It looks like agood adition

Bernard:
Privacy aspects?
You get the physical resolution if you don't use any constraints.
Pixels on the screen

Elad: Would it make sense to have a dedicated event?
I would prefer it on the controller. Processing might be efficient.
Youenn: Track is my preference but it's not blocking.
JIB: Prefer controller, but track wouldn't be blocking.

Mark: It's a  good idea. Sizes can change by the time a user calls applyConstraints. Also an issue with configurationchange event.

Conclusion: It's a good feature. There is consensus to add the properties and event in either track or controller.


### Element Capture (Elad Alon)
Capture an element/subtree of DOM ignoring the occlusions.

Google Meet has a feature called AddOns. 
Capture current tab, share an iframe with an external application. Prevent other elements to be drawn.
This has privacy advantages: no private messages / popups will be captured.

Case study: Add-ons SDK.  Marketplace where third-parties can integrate into Meet without much effort from Meet.

For example, without element capture, Figma-Meet integration would require a lot of integration.
Makes it easy for small players to integrate with large applications like Meet.

Feedback.
Trap sites are undeterred by detection. Answer: agreed.  
Tracking libraries can use this. Answer: such libraries avoid APIs that prompt.
It would be trivial to share. Answer: it's easier if the applications belong to the same company 

Summary:
Experiment in Chrome successful
Does not increase malicious actors' power to harm the user
Makes integrations for small third-parties

JIB:
Capture of occluded content. Mozilla objects because capturing occluded content.
Do things in the dark where things are occluded (e.g.,  opening private things that cannot be seen by the user), a tracking library might.
Would encourage to use cross-origin isolation, then a lot of concerns go away.
Restricting to getViewportMedia tracks would address most of the concerns.

Has there been any feedback from users about the occlusion?

Youenn: I would echo the concerns of Mozilla, and using cross-origin isolation would address most of them. There is an incentive to opt int by the third party.
Maybe add a CSS property to handle the occlusions.
Elad: Does not sound practical because some occluding things would draw.
