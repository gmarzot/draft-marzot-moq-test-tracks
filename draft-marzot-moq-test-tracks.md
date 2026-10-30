---
title: "MoQT Media Test Tracks"
abbrev: "moq-test-tracks"
category: info
docname: draft-marzot-moq-test-tracks-latest
submissiontype: IETF
number:
date:
consensus: false
v: 3
area: "Web and Internet Transport"
workgroup: "Media Over QUIC"
keyword:
 - moq
 - moqt
 - test
 - media
venue:
  group: "Media Over QUIC"
  type: "Working Group"
  mail: "moq@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/moq/"
  github: "gmarzot/draft-marzot-moq-test-tracks"
  latest: "https://gmarzot.github.io/draft-marzot-moq-test-tracks/draft-marzot-moq-test-tracks.html"

author:
 -
    fullname: Giovanni Marzot
    organization: Oracle
    email: giovanni.marzot@oracle.com

normative:
  MoQT: I-D.ietf-moq-transport
  MSF: I-D.ietf-moq-msf
  CMSF: I-D.ietf-moq-cmsf
  LOC: I-D.ietf-moq-loc
  MoQTimestamp: I-D.frindell-moq-timestamp

informative:
  MoQTest: I-D.afrind-moq-test
  MSF-EDITORS:
    title: "MOQT Streaming Format, editor's copy"
    target: https://moq-wg.github.io/msf/draft-ietf-moq-msf.html
    date: 2026-10-30
  LOCMAF: I-D.einarsson-moq-locmaf
  C4M: I-D.ietf-moq-c4m
  RFC6381:

--- abstract

This document specifies the moq-test-tracks protocol, a testing protocol for
Media over QUIC (MOQ) media implementations. Like moq-test, moq-test-tracks
utilizes the Track Namespace as parameters to the publisher. The parameters
describe a set of media tracks (catalog, packaging, codecs and delivery), so
that a publisher can generate media of a requested shape.

--- middle

# Introduction

moq-test {{MoQTest}} provides a testing framework for the MOQ transport,
using objects with synthetic payloads. moq-test-tracks extends the same
approach to media. It lets testers request a set of tracks with encoded
video and audio, a catalog, and a specific packaging. Catalog parsers,
packagers, players and relays can then be tested against media of a known
shape.

By leveraging the Track Namespace, moq-test-tracks lets testers specify the
content, catalog format, packaging, codecs, resolution, frame rate, group
duration, audio delivery and number of video tracks. Any publisher
that implements the protocol can produce them, without out-of-band
coordination. Each set of parameters has one shortest Track Namespace, which
publishers use, so subscribers requesting the same media share one set of
tracks, and results from different publishers can be compared.

# Track Namespace Parameters

The moq-test-tracks protocol utilizes the Track Namespace to convey the
parameters of a set of media tracks to the publisher. The Track Namespace
tuple fields are defined as follows.

* A Track Namespace can begin with a prefix: zero or more fields chosen by
  the deployment. This document does not interpret them, except that the
  last one identifies the clip when the content is clip ({{content}}). The
  tuple fields defined here follow the prefix and are numbered from 0.
* Tuple fields 0 through 8 are required. Tuple fields 9 through 13 are
  optional: the Track Namespace can end after any tuple field from 8
  onward, and each omitted tuple field takes the value given in its
  section.
* MoQT does not permit empty Track Namespace fields {{MoQT}}, so every
  tuple field that is present carries an explicit value.
* A field that does not apply to the set of tracks MUST be `_`. For example,
  the audio fields are `_` when there is no audio. An omitted field that
  does not apply is `_`.
* Field values use only the characters a-z, 0-9 and `_`, which the MoQT
  name representation renders verbatim ({{Section 8.8 of MoQT}}). Numeric
  fields are decimal integers without leading zeros.

Track Namespaces that differ only in tuple fields omitted from one and
spelled out with the same values in the other describe the same set of
tracks. A publisher that names a set of tracks itself, for example in
PUBLISH_NAMESPACE or PUBLISH, MUST omit every trailing tuple field whose
value equals the value it takes when omitted.

Future versions will use a different value for tuple field 0 and potentially
define a different set of parameters.

## Tuple Field 0: "moq_test_tracks_00"

This field indicates that the Track Namespace parameters are for the
moq-test-tracks protocol.

## Tuple Field 1: Content {#content}

This field specifies the picture and sound. The content determines how much
the media compresses, and so the size of the objects. Each value names a
class of content by what it lets a subscriber measure, not specific media.
The following values are defined:

* clock := A moving picture with a 440 Hz tone. The picture shows the
  capture time of each frame and a frame counter, and SHOULD show the group
  and object IDs. Once per second, a 1 kHz beep plays for 100 ms and the
  picture shows a marker for one frame, so that audio/video sync can be
  checked.
* bars := Static color bars with a 1 kHz tone. The media compresses to near
  the encoder's minimum.
* noise := Random noise in picture and sound. The encoder runs at the target
  bitrate ({{bitrates}}).
* clip := Content chosen by the publisher, such as a film or a recorded
  broadcast, encoded to the parameters of the namespace. This document does
  not specify any clip. Each deployment chooses the clips it serves and may
  add to them. The last field of the Track Namespace prefix identifies the
  clip. When a clip's license requires attribution, the picture SHOULD show
  it.

## Tuple Field 2: Catalog

This field specifies the catalog format published on the "catalog" track.
The following values are defined:

* msf := MSF catalog {{MSF}}, using the CMSF extensions {{CMSF}} for tracks
  with cmaf or locmaf packaging
* none := No catalog track

When the value is none, the subscriber learns the media configuration from
the Track Namespace and from the media itself ({{tracks}}).

## Tuple Field 3: Packaging

This field specifies the packaging of the media tracks. The following values
are defined:

* loc := LOC {{LOC}}
* cmaf := CMAF, as specified by {{CMSF}}
* locmaf := LOCMAF {{LOCMAF}}

Catalog none requires loc.

## Tuple Field 4: Video Codec

This field specifies the video codec. The following values are defined:

* h264 := H.264 High
* h264cb := H.264 Constrained Baseline
* h265 := H.265 Main
* av1 := AV1 Main
* none := No video track

Video is encoded without B-frames, so decode order is presentation order. The
level is the lowest that fits the height, frame rate and bitrate. When the
value is none, fields 5, 6, 10 and 13 are `_`.

## Tuple Field 5: Video Height

This field specifies the video height in pixels. The following values are
defined: 180, 360, 480, 540, 720, 1080, 1440 and 2160.

The width is the height multiplied by 16/9, rounded to the nearest even
number (854 for a height of 480).

## Tuple Field 6: Frame Rate

This field specifies the video frame rate in frames per second, as an integer
from 1 to 120. The values 23_976, 29_97 and 59_94 specify 24000/1001,
30000/1001 and 60000/1001.

## Tuple Field 7: Audio Codec

This field specifies the audio codec. The following values are defined:

* aac := AAC-LC
* opus := Opus with 20 ms frames
* none := No audio track

Audio is sampled at 48000 Hz. When the value is none, fields 8 and 11 are
`_`.

## Tuple Field 8: Audio Channels

This field specifies the number of audio channels. The following values are
defined:

* 1 := Mono
* 2 := Stereo

## Tuple Field 9: Init Data

This field specifies where the decoder initialization data of the media
tracks is carried. Each value corresponds to an entry type in the catalog's
Initialization Data List {{MSF}}; the track-property and object-property
types are in the MSF editor's copy {{MSF-EDITORS}}. The following values
are defined:

* catalog := In the catalog (type inline)
* track := In the VIDEO_CONFIG or AUDIO_CONFIG Track Property (type
  track-property)
* object := In the VIDEO_CONFIG or AUDIO_CONFIG Object Property on the first
  object of each group (type object-property)

With cmaf and locmaf packaging, the value is catalog, as CMSF requires for
CMAF headers {{CMSF}}. When tuple field 2 is none, the value is `_`, and the
initialization data is carried in the media itself ({{tracks}}).

When omitted, the value is catalog, or `_` when tuple field 2 is none.

## Tuple Field 10: Group Duration

This field specifies the interval between video key frames in milliseconds,
from 100 to 10000. Each video group begins with a key frame. The interval
MUST be a whole number of frames, for example 2000 at 30 frames per second
or 2002 at 29_97.

When omitted, the value is 2000, or 2002 when tuple field 6 is 23_976, 29_97
or 59_94.

## Tuple Field 11: Audio Delivery

This field specifies how audio frames are grouped and delivered. The
following values are defined:

* An integer from 1 to 100 := That many audio frames in each audio group,
  with each group on one stream ({{tracks}})
* dgram := Each audio frame is its own group and is sent in a datagram
  {{MoQT}}

dgram requires loc or cmaf packaging.

When omitted, the value is 1.

## Tuple Field 12: Duration

This field specifies the duration of the tracks in seconds, from 0 to 86400.
The value 0 means the tracks continue until unsubscribed.

When omitted, the value is 0.

## Tuple Field 13: Video Tracks {#video-tracks}

This field specifies how many video tracks the set contains, and how they
relate. The following values are defined:

* 1 := One video track
* altN, where N is from 2 to 7 := A ladder of N renditions of the same
  picture, for adaptive bitrate switching
* viewN, where N is from 2 to 16 := N views, each a different picture with
  the height and frame rate of tuple fields 5 and 6

When omitted, the value is 1.

In a ladder, the first rendition has the height of tuple field 5. Each
following rendition has the largest height defined for tuple field 5 that
is at most three quarters of the height of the rendition before it. N MUST
NOT exceed the number of renditions this yields from the height of tuple
field 5. For example, alt4 at a height of 1080 yields 1080, 720, 540 and
360. Every rendition has the frame rate of tuple field 6 and the bitrate of
its own height ({{bitrates}}).

With clock, bars or noise content, the views are distinct pictures of that
content. With clip content, the clip identifies a set of synchronized
views, and the publisher returns DOES_NOT_EXIST when the clip has fewer
than N views.

Values other than 1 require catalog msf.

# Tracks {#tracks}

Under the Track Namespace, the publisher publishes tracks with the following
Track Names:

* "catalog", unless tuple field 2 is none. This is the Track Name MSF
  reserves for the catalog {{MSF}}.
* "video", unless tuple field 4 is none. With more than one video track
  ({{video-tracks}}), the others are "video_1", "video_2" and so on, in
  ladder or view order.
* "audio", unless tuple field 7 is none

The "video", "video_N" and "audio" Track Names are defined by this
document.

The catalog describes the video and audio tracks with the parameters of the
namespace. This includes the codec string {{RFC6381}} for the exact
profile and level, and the bitrate ({{bitrates}}).
Each track in the catalog is live (isLive true) and has a targetLatency of
250 milliseconds {{MSF}}. No tuple field changes the target latency.

Each video group begins with a key frame and contains the frames up to the
next key frame. Each video frame is one object, and so is each audio frame.
With cmaf packaging, each object is one CMAF chunk ({{CMSF}}). Each group is
sent on one stream, as subgroup 0 {{MSF-EDITORS}}, except audio sent in
datagrams (tuple field 11).

When tuple field 2 is none:

* Video carries its parameter sets in the payload ({{LOC}}).
* With aac, the first object of each audio group carries the Audio Config
  property ({{LOC}}).

## Bitrates {#bitrates}

The target bitrate of each video track follows from its height:

| Height | Video kbps |
|-------:|-----------:|
| 180    | 200        |
| 360    | 600        |
| 480    | 1000       |
| 540    | 1500       |
| 720    | 2500       |
| 1080   | 4500       |
| 1440   | 9000       |
| 2160   | 16000      |

At frame rates above 30, the video bitrate is 1.5 times the value in the
table.

The target bitrate of the audio track follows from its codec and channels:

| Codec | Mono kbps | Stereo kbps |
|:------|----------:|------------:|
| aac   | 64        | 128         |
| opus  | 32        | 64          |

## Several Video Tracks {#video-sets}

When tuple field 13 specifies more than one video track:

* All video tracks share the frame rate and group duration. Frames captured
  at the same time have the same Group ID and Object ID in every video
  track, so a subscriber can switch between the tracks at any group
  boundary.
* The catalog gives every video track a label. In a ladder, the label is
  the height followed by "p", for example "720p". The views are numbered
  from 1 in Track Name order, and the label of a view is "view" followed by
  a space and its number, for example "view 2".
* With clock, bars or noise content, the picture of every video track shows
  its label, so that a switch between tracks is visible.
* In a ladder, the catalog gives all video tracks the same altGroup. Each
  view has an altGroup of its own.
* The set has one audio track, which has no altGroup.

## Timing

The capture time of a media object is the wall-clock time at which its
frame is captured, before it is encoded. The publisher captures frames on a
fixed schedule, so the capture time of each object follows from its Group ID
and Object ID. A publisher that encodes ahead of time and sends recorded
media uses each frame's scheduled time as its capture time.

Each media track carries the TIMESCALE and TIMESTAMP_MAPPING Track
Properties {{MoQTimestamp}}, without CLOCK_ID, so that timestamps are
wall-clock time:

* For video, the timescale is 1000 times the frame rate, or 24000, 30000
  and 60000 for 23_976, 29_97 and 59_94. The Object Multiplier is 1000, or
  1001 for those three frame rates.
* For audio, the timescale is 48000. The Object Multiplier is the
  number of samples per frame.
* The Group Multiplier is the number of objects per group multiplied by the
  Object Multiplier. The Base Group is the first Group ID published, and
  the Base Timestamp is the capture time of its first object.

An object whose capture time differs from its mapped timestamp carries
OBJECT_TIMESTAMP with the correction. A publisher that keeps to the schedule
sends no per-object timing.

In addition:

* With loc packaging, the Timestamp property ({{LOC}}) carries the capture
  time in microseconds since the Unix epoch, and the LOC Timescale property
  is not sent.
* With cmaf and locmaf packaging, the first object of each group contains a
  ProducerReferenceTimeBox (prft) with the capture time.

Latency is the time at which an object is received minus its capture time.
It includes encoding only when the publisher encodes live, so latency
results state how the publisher encoded the media. Measuring it requires the subscriber's clock to agree with the publisher's,
for example by running the subscriber on the publisher's host, or by
synchronizing both to a common time source.

# Publisher Behavior

If any namespace tuple field is not valid as defined in this document, or
describes tracks that the publisher does not support, a REQUEST_ERROR with
DOES_NOT_EXIST is returned. The reason phrase SHOULD identify the tuple
field. If producing the tracks would exceed the publisher's limits, a
REQUEST_ERROR with EXCESSIVE_LOAD is returned.

A publisher delivers a set of tracks in one of two ways. It sends
PUBLISH_NAMESPACE for the Track Namespace and then PUBLISH for each track,
so that relays receive the tracks before any subscriber asks for them. Or it
sends PUBLISH_NAMESPACE for the Track Namespace only, and relays forward
each SUBSCRIBE for a track in the namespace to it.

Upon receiving a valid SUBSCRIBE or FETCH for a moq-test-tracks track, the
publisher MUST publish objects according to the parameters of the namespace.

Upon receiving a SUBSCRIBE_TRACKS whose Track Namespace Prefix is a
moq-test-tracks Track Namespace, the publisher MUST use PUBLISH to send each
track in the namespace. If a PUBLISH is rejected, the publisher stops sending
objects for that track immediately.

A publisher MAY serve moq-test-tracks on demand. To do so, it sends
PUBLISH_NAMESPACE for a prefix whose last tuple field is "moq_test_tracks_00".
Relays route any SUBSCRIBE whose Track Namespace matches this prefix to the
publisher ({{Section 2.4.2 of MoQT}}).

* The publisher begins producing a set of tracks upon the first request for
  its Track Namespace.
* It serves later requests for the same Track Namespace from the same media.
  It MAY also serve a Track Namespace that describes the same set of tracks
  from the same media.
* It MAY stop producing the tracks when no subscriptions remain.

A publisher that serves clips on demand sends one PUBLISH_NAMESPACE for each
clip, with the clip's prefix. A subscriber can discover the clips a
deployment offers by sending SUBSCRIBE_NAMESPACE for the deployment's prefix
{{MoQT}}.

A Track Namespace that omits trailing tuple fields is a tuple prefix of the
Track Namespaces that spell them out. A relay that routes by Track Namespace
prefix can therefore forward a request for a longer Track Namespace to the
publisher of a shorter one, which returns DOES_NOT_EXIST unless it serves
the requested set of tracks. Likewise, a SUBSCRIBE_NAMESPACE for a
moq-test-tracks Track Namespace also matches the longer Track Namespaces
that extend it. A deployment that runs publishers of fixed sets of tracks
alongside an on-demand publisher SHOULD give them distinct prefixes.
Otherwise a relay can route a request that the on-demand publisher would
serve to a fixed publisher that cannot.

Ending a set of tracks is not an error. When the duration in tuple field 12
elapses, or when the publisher ends the tracks earlier, for example at a
limit of its own, the publisher ends each track gracefully {{MoQT}}:

* It marks the end of the track with the End of Track object status.
* It sends PUBLISH_DONE with status code TRACK_ENDED for each subscription
  to the track.
* If it sent PUBLISH_NAMESPACE for the Track Namespace of the set, it then
  sends PUBLISH_NAMESPACE_DONE.

The End of Track status takes the Location one past the last object of the
last group. It is sent on that group's stream before the stream ends. When
that stream has already ended, as is usual for the catalog track, the status
instead takes the first Location of the next group, on a new stream. With
dgram audio, the status is sent in a datagram and can be lost, so
subscribers rely on PUBLISH_DONE.

The publisher does not use GOAWAY to end tracks; GOAWAY asks peers to move
to a new session.

When audio is sent in datagrams, certain combinations of codec and channels
result in objects that cannot be transmitted below the maximum datagram
size. If the publisher detects this when handling the
request, it responds with REQUEST_ERROR. If it detects this when publishing
an object, it sends PUBLISH_DONE with an error code. The publisher MUST NOT
send such objects on a stream instead.

Media encoders are not deterministic, so object payloads differ between
publishers for the same Track Namespace. A subscriber that validates
moq-test-tracks media checks the parameters of the namespace rather than
payload bytes. These parameters are the codec configuration, frame rate,
group duration, objects per group, audio delivery and the video tracks of
the set.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Security Considerations

The subscriber can generate an asymmetric amount of work on the server, and
encoding media makes it larger than for moq-test. Servers SHOULD set limits
on values such as height, frame rate, duration, the number of video tracks
and the number of concurrent sets of tracks, to prevent resource exhaustion
attacks. Servers SHOULD serve all subscriptions to one Track Namespace from
a single set of tracks.

A publisher serving on demand SHOULD do so only through relays that
authorize subscribers for its prefix, for example with Common Access Tokens
{{C4M}}.

# IANA Considerations

This document has no IANA actions.

--- back

# Examples

The examples write the Track Namespace with its tuple fields separated by
"/", and wrap it after tuple field 0. The prefix is "openmoq/mtt", followed in
the clip example by a clip identifier.

720p30 H.264 with stereo AAC, MSF catalog, LOC packaging, initialization
data in the catalog:

~~~
openmoq/mtt/moq_test_tracks_00/
  clock/msf/loc/h264/720/30/aac/2
~~~

The same set of tracks with every optional tuple field spelled out. A
publisher names it with the shorter form above:

~~~
openmoq/mtt/moq_test_tracks_00/
  clock/msf/loc/h264/720/30/aac/2/catalog/2000/1/0/1
~~~

The "video" track of the first example, in the MoQT name representation
({{Section 8.8 of MoQT}}):

~~~
openmoq-mtt-moq_test_tracks_00-clock-msf-loc-h264-720-30-aac-2--video
~~~

1080p60 AV1 with Opus audio on datagrams, initialization data in Track
Properties:

~~~
openmoq/mtt/moq_test_tracks_00/
  clock/msf/loc/av1/1080/60/opus/2/track/2000/dgram
~~~

360p30 H.264 Constrained Baseline, initialization data on the first object
of each group:

~~~
openmoq/mtt/moq_test_tracks_00/
  clock/msf/loc/h264cb/360/30/aac/2/object
~~~

720p at 29.97 frames per second, CMAF packaging, no audio, for 10 minutes:

~~~
openmoq/mtt/moq_test_tracks_00/
  clock/msf/cmaf/h264/720/29_97/none/_/catalog/2002/_/600
~~~

Opus mono audio only, 10 frames per group, no catalog:

~~~
openmoq/mtt/moq_test_tracks_00/
  clock/none/loc/none/_/_/opus/1/_/_/10
~~~

A clip that the deployment identifies as "clip_a", as 1080p30 H.264 with
CMAF packaging:

~~~
openmoq/mtt/clip_a/moq_test_tracks_00/
  clip/msf/cmaf/h264/1080/30/aac/2
~~~

A three-rendition ladder from 1080p30 H.264. The video tracks are "video"
(1080p, 4500 kbps), "video_1" (720p, 2500 kbps) and "video_2" (540p, 1500
kbps), all in one altGroup:

~~~
openmoq/mtt/moq_test_tracks_00/
  clock/msf/loc/h264/1080/30/aac/2/catalog/2000/1/0/alt3
~~~

Four views of 720p30 H.264, as the tracks "video" ("view 1") through
"video_3" ("view 4"), each in its own altGroup:

~~~
openmoq/mtt/moq_test_tracks_00/
  clock/msf/loc/h264/720/30/aac/2/catalog/2000/1/0/view4
~~~

# Acknowledgments
{:numbered="false"}

This document builds on moq-test {{MoQTest}} by Alan Frindell. Thanks to
the OpenMOQ team for implementations and test deployments, and to members
of the Media over QUIC working group for discussion and review.
