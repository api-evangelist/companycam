---
name: Document a job site in CompanyCam
description: Create a project, add photos to it, and tag them — the core CompanyCam field-documentation flow.
api: openapi/companycam-openapi-original.yml
operations: [createProject, listProjects, createProjectPhoto, createPhotoTags, listProjectPhotos]
---

# Document a job site in CompanyCam

Use this to stand up a job/project and attach documented photos to it.

## Auth
Send `Authorization: Bearer <API_TOKEN>` on every request. Tokens come from
https://app.companycam.com/access_tokens or the OAuth 2.0 flow. API access
requires a Pro/Premium/Elite plan. Base URL: `https://api.companycam.com/v2`.

## Steps
1. (Optional) `listProjects` to check whether the project already exists before creating a duplicate — there is no idempotency key, so a repeated `createProject` makes a new project.
2. `createProject` with the site name and address. Capture the returned project `id`.
3. `createProjectPhoto` against that project `id` to add each photo. You may set a description at upload time.
4. `createPhotoTags` to tag each returned photo `id` for filtering later.
5. `listProjectPhotos` to confirm the photos landed on the project.

## Rules
- Rate limits: 240 GET/min, 100 write/min — back off and retry on HTTP 429.
- Errors come back as `{ "errors": ["message"] }` with the HTTP status carrying the machine signal (400/403/404/500). See errors/companycam-problem-types.yml.
- All ids are strings; all timestamps are unix-epoch seconds.
