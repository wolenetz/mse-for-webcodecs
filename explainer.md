# Media Source Extensions for WebCodecs

## Authors
* Matt Wolenetz @ Google

## Participate
* [Link to MSE Spec issue.](https://github.com/w3c/media-source/issues/184#issuecomment-720771445)
* [Link to Chromium prototype tracking issue.](https://crbug.com/1144908)

## Introduction

The [Media Source Extensions API (MSE)](https://www.w3.org/TR/media-source/) requires applications to provide fragments of containerized media
(such as fragmented MP4, WebM, or MP3) to be able to buffer and play that media.  If the application already has the media in a parsed and structured form,
it can cost extra latency and code complexity to package that media into a container and feed it to MSE.

As the web platform is evolving to give applications lower-level abstractions for efficiently encoding and decoding media via the
[WebCodecs API](https://github.com/WICG/web-codecs), MSE can use WebCodec's media structures to let applications more efficiently feed their media to MSE.

We propose additional MSE methods to provide web authors easy ability to buffer "containerless" media as an alternative to existing MSE methods that required
containerized, "muxed" media.

## Example Use Cases
* Simplifying use of MSE to play MPEG-TS muxed content on platforms not supporting MPEG-TS: Today, transmuxing into a supported container format is required,
and is done commonly in the browser.  With this proposal, such applications could instead just demux the MPEG-TS into WebCodecs Audio/VideoConfigs and EncodedAudio/VideoChunks
and feed those to MSE. This avoids both a remux step as well as a further demux of the supported container format within the MSE implementation.

* Low-latency streaming with a seekable buffer: By enabling containerless buffering of WebCodecs media configs and encoded media into MSE, improved low-latency MSE playback can be enabled (due to less complex buffering) while retaining existing seekability during playback of the buffered media.


## Proposed API

### Example 1: Buffering EncodedVideoChunks

```Javascript
// This example buffers some EncodedVideoChunks into MSE.

// App provides stream of encoded chunks (e.g., from encoder or from
// demux of an unsupported container format). We assume they are VP8
// in this example. Each invocation of newChunksCallback can be
// provided one or more VP8 EncodedVideoChunks.
async function fetchMoreEncodedChunks(newChunksCallback) { ... }

let mediaSource = new MediaSource();

// Later, following the normal steps to reach 'sourceopen':

// Create a SourceBuffer to hold VP8 EncodedVideoChunks.
// The VideoDecoderConfig provides information normally extracted from
// a containerized MSE initialization segment, so we just provide it
// instead of the usual mime-type and codecs string to addSourceBuffer().
let sourceBuffer = mediaSource.addSourceBuffer( {
    videoConfig: { codec: 'vp8' }
});

// Helper which demonstrates proposed promisified, async encoded chunk
// append usage.
async function bufferNewChunks(chunks) {
   // Asserting !sourceBuffer.updating is a good idea here.
   sourceBuffer.appendEncodedChunks(chunks)
       .then(() => fetchMoreEncodedChunks(bufferNewChunks));
}

// Start the fetching and buffering.
fetchMoreEncodedChunks(bufferNewChunks);
```

## Open Questions / Notes / Links

* `EncodedVideoChunk` duration is optional. But for interoperable buffering results, MSE depends upon coded frame duration. Should video chunks missing duration be rejected?

  * Update 2020-11-20: It's likely that MSE will reject an `EncodedVideoChunk` that does not have a duration attribute populated, to improve interoperability.
  
* `EncodedAudioChunk` has no duration attribute. Known encoded audio formats have presentation order matching decode order, so duration could be inferred from the timestamp delta between two successively appended `EncodedAudioChunk`s.
   But this suffers from similar latency and estimation issues as WebM SimpleBlocks do, especially for the chunks appended most recently or before an `abort()`, `changeType()`, etc call ends the current `coded frame group`. Should WebCodecs `EncodedAudioChunk`s get an optional duration attribute to improve this scenario?

  * Update 2020-11-20: It's likely that `EncodedAudioChunk`s will get an optional duration attribute to support this proposal. MSE could then reject an `EncodedAudioChunk` that does not have a duration attribute populated.

* None of the WebCodecs encoded chunks include a notion of decode timestamp. In regular MSE, decode timestamp helps detection of distinct bytestreams being appended without intervening abort(). Does decode timestamp make any sense for buffering WebCodecs media? Should we consider disallowing use of `sequence` `AppendMode` when buffering WebCodecs media?

  * Update 2020-11-20: Expected behavior clarification, especially for buffering out-of-order decode versus presentation content, will be forthcoming in this explainer as the prototype evolves to investigate feasible options.

* Should the new `addSourceBuffer` and `changeType` overloads be synchronous, or do they need to be new async methods instead? 

* Are there strong use cases for appending already-decoded audio or video?

  * Update 2020-11-20: This explainer is updated to move the decoded support out-of-scope. Relevant explainer text is moved to an addendum, and is marked out-of-scope and not proposed.

* Are there strong use cases for supporting more than one track in a `SourceBuffer` that is used to buffer WebCodecs media? The initial approach is to support precisely one track in such a SourceBuffer, since that is the most common usage of MSE currently, and keeps the API change simpler.

* Since it may be feasible to "sniff" whether or not a dictionary contains what is necessary to determine and differentiate a WebCodecs `VideoDecoderConfig` from an `AudioDecoderConfig`, is using a "dictionary of dictionaries" as the `SourceBufferConfig` type the most ergonomic shape for these API updates? This proposal attempts to keep the new API clear and simply implementable, though requires potentially more of the web application. Consulting TAG will occur before implementations of this proposal ship widely.

* [Link to GitHub repository.](https://github.com/wolenetz/mse-for-webcodecs/blob/main/explainer.md)

## Proposed IDL

```Javascript
dictionary SourceBufferConfig {
  // Precisely one of these must be populated to signal via addSourceBuffer or
  // changeType the intent to buffer the corresponding WebCodecs media.
  AudioDecoderConfig audioConfig;  // For appending EncodedAudioChunks.
  VideoDecoderConfig videoConfig;  // For appending EncodedVideoChunks.
};

// Additional method overload on the MediaSource interface:
SourceBuffer addSourceBuffer(SourceBufferConfig config);

// Note, the implementation MUST reject a sequence containing both audio and
// video chunks; it was more feasible to define this as a sequence of
// audio/video IDL unions in prototype of this feature in Chromium. 
typedef (sequence<(EncodedAudioChunk or EncodedVideoChunk)>
            or EncodedAudioChunk or EncodedVideoChunk) EncodedChunks;

// Additional method and a method overload on the SourceBuffer interface:
Promise<undefined> appendEncodedChunks(EncodedChunks chunks);

// This is expected to be useful to switch amongst encoded and containerized
// media in the same track in one SourceBuffer, or for simply changing the
// WebCodecs decoder configuration for upcoming encoded chunks.
undefined changeType(SourceBufferConfig config);
```

## Addendum - (Out-of-Scope of this explainer, not proposed): Supporting Decoded Frames?

It is unclear at the moment if it makes sense to also support buffering
__decoded__ WebCodecs AudioFrames and VideoFrames in MSE versus using
other low-latency streaming (but not seeking) APIs. This addendum
describes briefly a possible use case and IDL, though should not be
understood as a commitment to prototype nor standardize at this
time. The focus of the proposal in this explainer is squarely on
buffering WebCodecs __encoded__ media in MSE.

### Possible use case (out-of-scope, not proposed)

* Buffering already decoded PCM audio: As an alternative to adding a "WAV" or similar bytestream format, if raw decoded audio could be buffered instead, this
could help resolve this [longstanding request of MSE](https://github.com/w3c/media-source/issues/55).

### Possible IDL (out-of-scope, not proposed)

```Javascript
enum DecodedMediaType {
  "audio-frames",
  "video-frames"
};

// An update to proposed SourceBufferConfig:
{
  ...
  DecodedMediaType decodedMediaType;  // For appending {Audio,Video}Frames.
};

typedef (sequence<(AudioFrame or VideoFrame)>
            or AudioFrame or VideoFrame) DecodedFrames;

// Additional method on the SourceBuffer interface:
Promise<undefined> appendDecodedFrames(DecodedFrames frames);
```
