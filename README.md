# ComfyUI-CloudArchive

ComfyUI-CloudArchive is a plugin that monitors the ComfyUI output directory and automatically archives (saves) generated files to Amazon S3 or S3-compatible storage.

## Features

- Monitors ComfyUI's output directory and detects new files (including subdirectories)
- Automatically uploads detected files to S3 or S3-compatible storage
- Intelligently waits until files are completely written (detects file size stabilization)
- Preserves the internal structure of the output directory when uploading
- Generates a unique session ID at startup for folder organization in S3
- RESTful API endpoints for status checking and control
- Easy configuration via environment variables

## Installation

1. Clone this repository into ComfyUI's custom nodes directory:

```bash
cd /path/to/ComfyUI/custom_nodes
git clone https://github.com/olduvai-jp/ComfyUI-CloudArchive.git
```

2. Install the required Python packages:

```bash
pip install boto3 watchdog
```

## Configuration

Set the following environment variables:

- `S3_ACCESS_KEY_ID` (required): Access key ID for the storage service
- `S3_SECRET_ACCESS_KEY` (required): Secret access key for the storage service
- `S3_REGION` (optional): Region for the storage service (default: us-east-1)
- `S3_BUCKET` (required): Bucket name for uploads
- `S3_PREFIX` (optional): Prefix within the bucket (default: comfyui-outputs)
- `S3_ENDPOINT_URL` (optional): URL for S3-compatible endpoint (MinIO, Wasabi, DigitalOcean Spaces, etc.)

Example environment variable setup (Linux/Mac):

```bash
export S3_ACCESS_KEY_ID=your_access_key
export S3_SECRET_ACCESS_KEY=your_secret_key
export S3_REGION=ap-northeast-1
export S3_BUCKET=your-bucket-name
export S3_PREFIX=comfyui/outputs
export S3_ENDPOINT_URL=https://minio.example.com
```

Windows:

```cmd
set S3_ACCESS_KEY_ID=your_access_key
set S3_SECRET_ACCESS_KEY=your_secret_key
set S3_REGION=ap-northeast-1
set S3_BUCKET=your-bucket-name
set S3_PREFIX=comfyui/outputs
set S3_ENDPOINT_URL=https://minio.example.com
```

### Using S3-Compatible Endpoints

This plugin supports not only Amazon S3 but also S3-compatible storage services. It can be used with services such as:

- [MinIO](https://min.io/)
- [Wasabi](https://wasabi.com/)
- [DigitalOcean Spaces](https://www.digitalocean.com/products/spaces)
- [Backblaze B2](https://www.backblaze.com/b2/cloud-storage.html)
- [Scaleway Object Storage](https://www.scaleway.com/en/object-storage/)
- Other S3-compatible services

To use an S3-compatible service, set the `S3_ENDPOINT_URL` environment variable to the endpoint URL of the service.

#### MinIO Example:

```bash
export S3_ACCESS_KEY_ID=your_minio_access_key
export S3_SECRET_ACCESS_KEY=your_minio_secret_key
export S3_BUCKET=your-minio-bucket
export S3_ENDPOINT_URL=http://minio.example.com:9000
```

#### Wasabi Example:

```bash
export S3_ACCESS_KEY_ID=your_wasabi_access_key
export S3_SECRET_ACCESS_KEY=your_wasabi_secret_key
export S3_REGION=us-east-1
export S3_BUCKET=your-wasabi-bucket
export S3_ENDPOINT_URL=https://s3.wasabisys.com
```

#### DigitalOcean Spaces Example:

```bash
export S3_ACCESS_KEY_ID=your_spaces_key
export S3_SECRET_ACCESS_KEY=your_spaces_secret
export S3_REGION=nyc3
export S3_BUCKET=your-space-name
export S3_ENDPOINT_URL=https://nyc3.digitaloceanspaces.com
```

## Usage

### Automatic Startup

The plugin starts automatically when ComfyUI is launched and monitors the default output directory (`ComfyUI/output`).

### API Endpoints

The following API endpoints are available:

#### Status Check

```
GET /cloud-sync/status
```

Example response:

```json
{
  "running": true,
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "total_files": 10,
  "uploaded_files": 8,
  "failed_files": 0,
  "last_upload_time": "2023-04-01T12:34:56",
  "errors": [],
  "recent_uploads": [
    {
      "file_name": "image_01.png",
      "s3_key": "comfyui-outputs/550e8400-e29b-41d4-a716-446655440000/image_01.png",
      "upload_time": "2023-04-01T12:34:56",
      "size_bytes": 1234567
    }
  ]
}
```

#### Start Monitoring

```
POST /cloud-sync/start
```

#### Stop Monitoring

```
POST /cloud-sync/stop
```

#### Manual Upload

```
POST /cloud-sync/upload
```

Request body:

```json
{
  "file_path": "/path/to/file.png"
}
```

## S3 Directory Structure

Uploaded files are stored in S3 with the following structure:

```
{S3_PREFIX}/{SESSION_ID}/{RELATIVE_PATH}
```

Example:

```
comfyui-outputs/550e8400-e29b-41d4-a716-446655440000/image_01.png
comfyui-outputs/550e8400-e29b-41d4-a716-446655440000/subfolder/image_02.png
```

The internal structure of the output directory (including subdirectories) is preserved.

## Troubleshooting

### Environment Variables Not Set

If the required environment variables are not set, error messages will be logged and displayed in the `errors` field of the `/cloud-sync/status` endpoint.

### Upload Failures

If an upload fails, error messages will be logged and displayed in the `errors` field of the `/cloud-sync/status` endpoint. The `failed_files` counter will also increase.

## License

MIT