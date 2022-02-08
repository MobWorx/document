# Data Signals

## General Shape (Client -> Server)

Only the payload field is subject to change across the apis

Not all attributes are included. We may add those in the later versions.

```json
{
	action: String // for aws apigateway routing
	version: Int // our protocol version
	id: String // UUID
	timestamp: String // milliseconds since 1970. Captured when client send this message
	numbytes: Int // payload size
	payload: {
		playlist: { // master
			id: String // save key, also use as name
			version: Int // hls version
		}
		variant: { // stream, #EXT-X-STREAM-INF
			id: String // save key, also use as name
			codecs: String
			bandwidth: Int
			audio: String
			version: Int
			targetDuration: Int
			targetPartDuration: Double
		}
		rendition: { // media, #EXT-X-MEDIA, currently we assume only master contain it
			id: String // save key, also use as name
			type: String
			groupId: String
			name: String
			language: String
			isDefault: Bool
			autoSelect: Bool
			targetDuration: Int
			targetPartDuration: Double
		}
		segment: {
			id: String // save key, also use as name
			sequence: Int // segment seq num
			duration: Double
			discontinuity: Bool
			programDateTime: String // milliseconds since 1970, server will parse it
			map: { // Media Initialization Section
				id: String // save key, name for init section
				data: String // binary data in base64 encode
            }
			data: String // binary data in base64 encode
		}
		part: {
			id: String // save key, also use as name
			sequence: Int // part seg num
			duration: Double
			independent: Bool
			gap: Bool
			data: String // binary data in base64 encode
		}
	}
}

```

## API V1

For version 1, we assume that each message is an update of one resource. Ex variant, rendition, segment, part. So we cannot upload an audio segment and a video segment at the same time. There is one exception for this. Client can upload segment->map->data along with segment->data or part->data.

> ? mark represent the field is optional

### updateVariant

- Create playlist object in server if the playlist->id doesn’t existed
- Append another variant stream if the variant->id doesn’t existed

```json
{
	action: “updateVariant”
	version: 1
	id: String // UUID
	timestamp: String // milliseconds since 1970. Captured when client send this message
	numbytes: Int // payload size
	payload: {
		playlist: { // master
			id: String // save key, also use as name
			version: String // hls version
		}
		variant: {
			id: String // save key, also use as name
			codecs: String
			bandwidth: Int
			audio: String
			version: String
			targetDuration: Int
			targetPartDuration: Double
		}
		segment: {
			map: {
				id: String // save key, name for init section
				data: String // binary data in base64 encode
            },
		}
	}
}
```

### updateRendition

```json
{
	action: “updateRendition”
	version: 1
	id: String // UUID
	timestamp: String // milliseconds since 1970. Captured when client send this message
	numbytes: Int // payload size
	payload: {
		playlist: { // master
			id: String // save key, also use as name
		}
		rendition: { // media, #EXT-X-MEDIA, currently we assume only master contain it
			id: String // save key, also use as name
			type: String
			groupId: String
			name: String
			language: String
			isDefault: Bool
			autoSelect: Bool
			targetDuration: Int
			targetPartDuration: Double
		}
		segment: {
			map: {
				id: String // save key, name for init section
				data: String // binary data in base64 encode
            }
		}
	}
}

```

### updateSegment

```json
{
	action: “updateSegment”
	version: 1
	id: String // UUID
	timestamp: String // milliseconds since 1970. Captured when client send this message
	numbytes: Int // payload size
	payload: {
		playlist: { // master
			id: String // save key, also use as name
		}
		variant: { // will == nil if rendition field exist
			id: String // save key, also use as name
		}
		rendition: { // will == nil if variant field exist
			id: String // save key, also use as name
		}
		segment: {
			id: String // save key, also use as name
			sequence: Int // segment seq num
			duration: Double
			discontinuity: Bool?
			programDateTime: String? // milliseconds since 1970, server will parse it
			map: {
				id: String // save key, name for init section
				data: String // binary data in base64 encode
            }?
			data: String // binary data in base64 encode
		}
	}
}

```

### updatePart

```json
{
	action: “updatePart”
	version: 1
	id: String // UUID
	timestamp: String // milliseconds since 1970. Captured when client send this message
	numbytes: Int // payload size
	payload: {
		playlist: { // master
			id: String // save key, also use as name
		}
		variant: { // will == nil if rendition field exist
			id: String // save key, also use as name
		}
		rendition: { // will == nil if variant field exist
			id: String // save key, also use as name
			type: String
		}
		segment: {
			id: String // save key, also use as name
			sequence: Int // segment seq num
			duration: Double
			discontinuity: Bool?
			programDateTime: String? // milliseconds since 1970, server will parse it
			map: {
				id: String // save key, name for init section
				data: String // binary data in base64 encode
            }?
		}
		part: {
			id: String // save key, also use as name
			sequence: Int // part seg num
			duration: Double
			independent: Bool?
			gap: Bool?
			data: String // binary data in base64 encode
		}
	}
}

```