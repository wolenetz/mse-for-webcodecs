Media Source Extensions for WebCodecs - Security and Privacy Questionnaire

This document answers the [W3C Security and Privacy self-review Questionnaire](https://w3.org/TR/security-privacy-questionnaire/) for the Media Source Extensions for WebCodecs feature specification.

Last Update: 2020-11-23

**2.1 What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

This feature may expose additional information about what media a user agent is capable of buffering, decoding and rendering using Media Source Extensions.
Such information is mostly already available via other APIs, especially [WebCodecs](https://wicg.github.io/web-codecs/), [Media Source Extensions](https://www.w3.org/TR/media-source/), WebRTC and MediaCapabilities.
Like WebCodecs itself, this feature may subtly expose more information or make it easier to obtain the same information (e.g., this feature may expose timing information about how
long media takes to decode and/or render).

This is a necessary part of enabling a more efficient mechanism for apps to provide containerless media to be buffered and played with Media Source Extensions.

**2.2 Is this specification exposing the minimum amount of information necessary to power the feature?

Yes. It is adding a feature-detectable ability for apps to provide WebCodecs data structures (EncodedAudioChunks and EncodedVideoChunks, perhaps also later decoded AudioFrames and VideoFrames too)
for more efficient buffering and seekable playback of media.

**2.3 How does this specification deal with personal information or personally-identifiable information or information derived thereof?

There is no PII involved.

**2.4 How does this specification deal with sensitive information?

As with the existing Media Source Extensions, this feature avoids exposing such information, such as user's hardware, operating system, etc.

**2.5 Does this specification introduce new state for an origin that persists across browsing sessions?

No

**2.6 What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?

As noted above, it may indirectly expose what the underlying platform is capable of (which is largely possible already), but such information would be consistent across
origins and would not be related to user configuration.

**2.7 Does this specification allow an origin access to sensors on a user’s device

No.

**2.8 What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

As noted above, it exposes information about what a platform is capable of in terms of decoding and rendering, which is already possible via other media APIs such
as WebCodecs, Media Source Extensions, WebRTC and MediaCapabilities, but may subtly expose more information such as more specific timing information.

**2.9 Does this specification enable new script execution/loading mechanisms?

No.

**2.10 Does this specification allow an origin to access other devices?

No.

**2.11 Does this specification allow an origin some measure of control over a user agent’s native UI?

Not directly. As it is an additional feature for Media Source Extensions, it will be used in conjunction with an extended HTMLMediaElement, which may itself have
integrations already with a user agent's native UI (such as MediaSession and FullScreen media playback integrations). But this feature does not itself change what
control over the native UI is already provided via those other specifications.

**2.12 What temporary identifiers might this this specification create or expose to the web?

None.

**2.13 How does this specification distinguish between behavior in first-party and third-party contexts?

The specificiation does not distinguish between 1st and 3rd party.

**2.14 How does this specification work in the context of a user agent’s Private Browsing or "incognito" mode?

The specification behaves in the same way.

**2.15 Does this specification have a "Security Considerations" and "Privacy Considerations" section?

Not yet, as the specification for this new feature for the existing Media Source Extensions specification hasn't been formally written.
When it is written, it should carefully consider what new information is available that was not already available with existing media APIs.
Also, it's highly likely that such section will be added to the overall Media Source Extensions specification, as the the V2 version of that specification
is where such new features like this "Media Source Extensions for WebCodecs" will be specified;
this questionnaire led to us filing https://github.com/w3c/media-source/issues/261 to track getting these 2 sections added to that main specification.

**2.16 Does this specification allow downgrading default security characteristics?

No.

**2.17 What should this questionnaire have asked?

No comment.
