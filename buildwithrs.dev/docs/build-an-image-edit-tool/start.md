---
sidebar_position: 1
---

# The Image Edit Tool

So I want to build an real world app that helps me to learn more about rust web programming.

And for this image edit project, it will support following functions:

- To help users upload images.
- To add watermark to the images.
- To resize the images.
- Let user corp the images.
- User can compress the images into another format.
- Let user can download the images after the above operations is done.

## The API Design

Since we have defined the goals of this project. then we can degisn the APIs according our requirments.

For your Rust-based backend image editing service, I'll design a RESTful API using standard HTTP methods. The backend will handle image uploads and provide endpoints for each editing feature (watermark, resize, crop, compress). To keep it simple and scalable:

- Images are uploaded and stored temporarily on the server (e.g., in a directory or database; you can use crates like `actix-web` for the web framework, `image` for processing, and `uuid` for IDs).
- Each uploaded image gets a unique ID.
- Edits are non-destructive: They create a new version of the image and return a new ID or direct URL to the result. This allows chaining operations (e.g., resize then watermark).
- Authentication is omitted for simplicity; add JWT or API keys if needed.
- Error handling: Standard HTTP status codes (e.g., 400 for bad requests, 404 for missing images, 500 for server errors) with JSON error bodies.
- Responses: JSON for metadata, with image data served via GET endpoints.

Assume the base URL is `/api/images`. All endpoints require appropriate headers (e.g., `Content-Type: application/json` for bodies).

### Endpoints and Requests

I'll describe each endpoint with:

- **Method and Path**
- **Description**
- **Request Parameters/Body**
- **Response**
- **Example Request (cURL-style)**

Use tables for structured details where helpful.

#### 1. Upload Image

This endpoint allows users to upload an image file, which is stored on the server and assigned an ID.

- **Method and Path**: `POST /api/images/upload`
- **Description**: Uploads an image file (supports JPEG, PNG, etc.). Returns the image ID and a URL to access it.
- **Request**:
  - Body: Multipart form data.
  - Required fields:

    | Field Name | Type   | Description                  |
    |------------|--------|------------------------------|
    | file      | binary | The image file to upload.    |

- **Response** (200 OK):
  - JSON body: `{ "image_id": "uuid-string", "url": "http://server/api/images/{image_id}" }`
- **Example Request**:

  ```sh
  curl -X POST http://localhost:8080/api/images/upload \
  -F "file=@/path/to/image.jpg"
  ```

#### 2. Get Image
Retrieve the original or edited image by ID.

- **Method and Path**: `GET /api/images/{image_id}`
- **Description**: Serves the image binary data.
- **Request**:
  - Path Params:

    | Param     | Type   | Description                  |
    |-----------|--------|------------------------------|
    | image_id | string | Unique ID of the image.      |

- **Response** (200 OK):
  - Binary image data (with `Content-Type: image/*` header).
- **Example Request**:

  ```sh
  curl -X GET http://localhost:8080/api/images/123e4567-e89b-12d3-a456-426614174000 --output edited.jpg
  ```

#### 3. Add Watermark

Apply a text or image watermark to the image.

- **Method and Path**: `POST /api/images/{image_id}/watermark`
- **Description**: Adds a watermark (text-based for simplicity; extend to image if needed). Creates a new image version.
- **Request**:
  - Path Params:

    | Param     | Type   | Description                  |
    |-----------|--------|------------------------------|
    | image_id | string | ID of the source image.      |

  - Body: JSON.

    | Field     | Type   | Required | Description                          |
    |-----------|--------|----------|--------------------------------------|
    | text     | string | Yes     | Watermark text (e.g., "Copyright"). |
    | position | string | No      | Position: "top-left", "center", etc. (default: "bottom-right"). |
    | opacity  | float  | No      | Opacity (0.0-1.0, default: 0.5).   |
    | font_size| integer| No      | Font size in pixels (default: 20).  |

- **Response** (200 OK):
  - JSON body: `{ "new_image_id": "uuid-string", "url": "http://server/api/images/{new_image_id}" }`
- **Example Request**:

  ```sh
  curl -X POST http://localhost:8080/api/images/123e4567-e89b-12d3-a456-426614174000/watermark \
  -H "Content-Type: application/json" \
  -d '{"text": "My Watermark", "position": "center", "opacity": 0.7, "font_size": 30}'
  ```

#### 4. Resize Image

Resize the image to specified dimensions.

- **Method and Path**: `POST /api/images/{image_id}/resize`
- **Description**: Resizes the image while maintaining aspect ratio if only one dimension is provided. Creates a new version.
- **Request**:
  - Path Params:

    | Param     | Type   | Description                  |
    |-----------|--------|------------------------------|
    | image_id | string | ID of the source image.      |

  - Body: JSON.

    | Field   | Type    | Required | Description                          |
    |---------|---------|----------|--------------------------------------|
    | width  | integer | No      | New width in pixels (required if height omitted). |
    | height | integer | No      | New height in pixels (required if width omitted). |
    | maintain_aspect | boolean | No | Maintain ratio? (default: true).    |

- **Response** (200 OK):
  - JSON body: `{ "new_image_id": "uuid-string", "url": "http://server/api/images/{new_image_id}" }`
- **Example Request**:

  ```sh
  curl -X POST http://localhost:8080/api/images/123e4567-e89b-12d3-a456-426614174000/resize \
  -H "Content-Type: application/json" \
  -d '{"width": 800, "height": 600, "maintain_aspect": true}'
  ```

#### 5. Crop Image

Crop a portion of the image.

- **Method and Path**: `POST /api/images/{image_id}/crop`
- **Description**: Crops based on coordinates and dimensions. Creates a new version.
- **Request**:
  - Path Params:

    | Param     | Type   | Description                  |
    |-----------|--------|------------------------------|
    | image_id | string | ID of the source image.      |

  - Body: JSON.

    | Field   | Type    | Required | Description                          |
    |---------|---------|----------|--------------------------------------|
    | x      | integer | Yes     | Starting x-coordinate (top-left=0). |
    | y      | integer | Yes     | Starting y-coordinate.              |
    | width  | integer | Yes     | Crop width in pixels.               |
    | height | integer | Yes     | Crop height in pixels.              |

- **Response** (200 OK):
  - JSON body: `{ "new_image_id": "uuid-string", "url": "http://server/api/images/{new_image_id}" }`
- **Example Request**:

  ```sh
  curl -X POST http://localhost:8080/api/images/123e4567-e89b-12d3-a456-426614174000/crop \
  -H "Content-Type: application/json" \
  -d '{"x": 100, "y": 100, "width": 400, "height": 300}'
  ```

#### 6. Compress Image

Compress the image to reduce file size.

- **Method and Path**: `POST /api/images/{image_id}/compress`
- **Description**: Applies lossy compression (e.g., for JPEG). Creates a new version.
- **Request**:
  - Path Params:

    | Param     | Type   | Description                  |
    |-----------|--------|------------------------------|
    | image_id | string | ID of the source image.      |

  - Body: JSON.

    | Field   | Type    | Required | Description                          |
    |---------|---------|----------|--------------------------------------|
    | quality| integer | No      | Compression quality (0-100, default: 80). |
    | format | string  | No      | Output format: "jpeg", "png" (default: original). |

- **Response** (200 OK):
  - JSON body: `{ "new_image_id": "uuid-string", "url": "http://server/api/images/{new_image_id}" }`
- **Example Request**:

  ```sh
  curl -X POST http://localhost:8080/api/images/123e4567-e89b-12d3-a456-426614174000/compress \
  -H "Content-Type: application/json" \
  -d '{"quality": 70, "format": "jpeg"}'
  ```

### Implementation Notes for Rust

- Use `axum` for the server.
- For image processing: Crate `image` for resize/crop/watermark/compress.
- Storage: Save files in a `./uploads` dir with UUID filenames.
- Cleanup: Implement a mechanism to delete old images (e.g., TTL).
- Security: Validate file types, sizes, and inputs to prevent attacks.
- Chaining: Frontend can call endpoints sequentially using the returned `new_image_id`.