# OpenAPI Annotation Template

Standard JSDoc annotation pattern for Sigma API routes.
Use this as a reference when writing `@openapi` blocks on new endpoints.

---

## GET — List (no path param)

```js
/**
 * @openapi
 * /api/{resource}:
 *   get:
 *     summary: List all {resources}
 *     operationId: list{Resources}
 *     description: |
 *       `GET /api/{resource}`
 *
 *       {What this endpoint returns and any ordering/filtering behaviour.}
 *     tags: [{Tag}]
 *     security:
 *       - sessionAuth: []
 *       - patAuth: []
 *     responses:
 *       200:
 *         description: List of {resources}
 *         content:
 *           application/json:
 *             example:
 *               - id: clx1234abcd
 *                 name: Example Name
 *             schema:
 *               type: array
 *               items:
 *                 type: object
 *                 required: [id, name, createdAt, updatedAt]
 *                 properties:
 *                   id:
 *                     type: string
 *                     description: Unique identifier (cuid)
 *                     example: clx1234abcd
 *                   name:
 *                     type: string
 *                     description: Display name
 *                     example: Example Name
 *                   createdAt:
 *                     type: string
 *                     format: date-time
 *                     description: Timestamp when the record was created
 *                     example: "2026-04-22T08:00:00.000Z"
 *                   updatedAt:
 *                     type: string
 *                     format: date-time
 *                     description: Timestamp when the record was last modified
 *                     example: "2026-04-22T08:00:00.000Z"
 *       401:
 *         $ref: '#/components/responses/Unauthorized'
 *       403:
 *         $ref: '#/components/responses/Forbidden'
 *       500:
 *         $ref: '#/components/responses/InternalError'
 */
```

---

## POST — Create

```js
/**
 * @openapi
 * /api/{resource}:
 *   post:
 *     summary: Create a new {resource}
 *     operationId: create{Resource}
 *     description: |
 *       `POST /api/{resource}`
 *
 *       {What this endpoint does. List any constraints.}
 *
 *       **Constraints:**
 *       - {constraint 1}
 *       - {constraint 2}
 *     tags: [{Tag}]
 *     security:
 *       - sessionAuth: []
 *       - patAuth: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           example:
 *             name: Example Name
 *           schema:
 *             type: object
 *             required: [name]
 *             properties:
 *               name:
 *                 type: string
 *                 description: Display name — must be unique, max 255 characters
 *                 example: Example Name
 *     responses:
 *       201:
 *         description: "{Resource} created successfully"
 *         content:
 *           application/json:
 *             example:
 *               id: clx1234abcd
 *               name: Example Name
 *               createdAt: "2026-04-22T08:00:00.000Z"
 *               updatedAt: "2026-04-22T08:00:00.000Z"
 *             schema:
 *               type: object
 *               required: [id, name, createdAt, updatedAt]
 *               properties:
 *                 id:
 *                   type: string
 *                   description: Unique identifier (cuid)
 *                   example: clx1234abcd
 *                 name:
 *                   type: string
 *                   description: Display name
 *                   example: Example Name
 *                 createdAt:
 *                   type: string
 *                   format: date-time
 *                   example: "2026-04-22T08:00:00.000Z"
 *                 updatedAt:
 *                   type: string
 *                   format: date-time
 *                   example: "2026-04-22T08:00:00.000Z"
 *       400:
 *         $ref: '#/components/responses/BadRequest'
 *       401:
 *         $ref: '#/components/responses/Unauthorized'
 *       403:
 *         $ref: '#/components/responses/Forbidden'
 *       500:
 *         $ref: '#/components/responses/InternalError'
 */
```

---

## GET — Single by ID

```js
/**
 * @openapi
 * /api/{resource}/{id}:
 *   get:
 *     summary: Get a {resource} by ID
 *     operationId: get{Resource}ById
 *     description: |
 *       `GET /api/{resource}/{id}`
 *
 *       {What this returns.}
 *     tags: [{Tag}]
 *     security:
 *       - sessionAuth: []
 *       - patAuth: []
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         description: "{Resource} ID (cuid)"
 *         example: clx1234abcd
 *         schema:
 *           type: string
 *           example: clx1234abcd
 *     responses:
 *       200:
 *         description: "{Resource} details"
 *         content:
 *           application/json:
 *             example:
 *               id: clx1234abcd
 *               name: Example Name
 *               createdAt: "2026-04-22T08:00:00.000Z"
 *               updatedAt: "2026-04-22T08:00:00.000Z"
 *             schema:
 *               type: object
 *               required: [id, name, createdAt, updatedAt]
 *               properties:
 *                 id:
 *                   type: string
 *                   description: Unique identifier (cuid)
 *                   example: clx1234abcd
 *                 name:
 *                   type: string
 *                   description: Display name
 *                   example: Example Name
 *                 createdAt:
 *                   type: string
 *                   format: date-time
 *                   example: "2026-04-22T08:00:00.000Z"
 *                 updatedAt:
 *                   type: string
 *                   format: date-time
 *                   example: "2026-04-22T08:00:00.000Z"
 *       401:
 *         $ref: '#/components/responses/Unauthorized'
 *       403:
 *         $ref: '#/components/responses/Forbidden'
 *       404:
 *         $ref: '#/components/responses/NotFound'
 *       500:
 *         $ref: '#/components/responses/InternalError'
 */
```

---

## PATCH/PUT — Update by ID

```js
/**
 * @openapi
 * /api/{resource}/{id}:
 *   patch:
 *     summary: Update a {resource}
 *     operationId: update{Resource}
 *     description: |
 *       `PATCH /api/{resource}/{id}`
 *
 *       {What fields can be updated.}
 *     tags: [{Tag}]
 *     security:
 *       - sessionAuth: []
 *       - patAuth: []
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         description: "{Resource} ID (cuid)"
 *         example: clx1234abcd
 *         schema:
 *           type: string
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           example:
 *             name: Updated Name
 *           schema:
 *             type: object
 *             properties:
 *               name:
 *                 type: string
 *                 description: New display name
 *                 example: Updated Name
 *     responses:
 *       200:
 *         description: "{Resource} updated successfully"
 *         content:
 *           application/json:
 *             example:
 *               id: clx1234abcd
 *               name: Updated Name
 *               updatedAt: "2026-04-24T10:30:00.000Z"
 *             schema:
 *               type: object
 *               required: [id, name, updatedAt]
 *               properties:
 *                 id:
 *                   type: string
 *                   example: clx1234abcd
 *                 name:
 *                   type: string
 *                   example: Updated Name
 *                 updatedAt:
 *                   type: string
 *                   format: date-time
 *                   example: "2026-04-24T10:30:00.000Z"
 *       400:
 *         $ref: '#/components/responses/BadRequest'
 *       401:
 *         $ref: '#/components/responses/Unauthorized'
 *       403:
 *         $ref: '#/components/responses/Forbidden'
 *       404:
 *         $ref: '#/components/responses/NotFound'
 *       500:
 *         $ref: '#/components/responses/InternalError'
 */
```

---

## DELETE — Delete by ID

```js
/**
 * @openapi
 * /api/{resource}/{id}:
 *   delete:
 *     summary: Delete a {resource}
 *     operationId: delete{Resource}
 *     description: |
 *       `DELETE /api/{resource}/{id}`
 *
 *       Permanently deletes the {resource}. This action cannot be undone.
 *     tags: [{Tag}]
 *     security:
 *       - sessionAuth: []
 *       - patAuth: []
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         description: "{Resource} ID (cuid)"
 *         example: clx1234abcd
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: "{Resource} deleted successfully"
 *         content:
 *           application/json:
 *             example:
 *               deleted: true
 *               id: clx1234abcd
 *             schema:
 *               type: object
 *               required: [deleted, id]
 *               properties:
 *                 deleted:
 *                   type: boolean
 *                   description: Always true when the operation succeeds
 *                   example: true
 *                 id:
 *                   type: string
 *                   description: ID of the deleted {resource}
 *                   example: clx1234abcd
 *       401:
 *         $ref: '#/components/responses/Unauthorized'
 *       403:
 *         $ref: '#/components/responses/Forbidden'
 *       404:
 *         $ref: '#/components/responses/NotFound'
 *       500:
 *         $ref: '#/components/responses/InternalError'
 */
```

---

## Field Type Reference

| Data type | `type` | `format` | Example |
|---|---|---|---|
| cuid / ID | `string` | — | `clx1234abcd` |
| URL | `string` | `uri` | `https://...` |
| Datetime | `string` | `date-time` | `"2026-04-22T08:00:00.000Z"` |
| Boolean | `boolean` | — | `true` |
| Integer | `integer` | — | `15` |
| Enum integer | `integer` + `enum: [...]` | — | `enum: [1, 5, 10, 15, 30, 365]` |
| Nullable field | add `nullable: true` | — | — |

## Error Response Reference

All error responses use `$ref` to shared components defined in `generate-openapi.ts`:

| Status | `$ref` | When to include |
|---|---|---|
| 400 | `'#/components/responses/BadRequest'` | Endpoints that validate request input |
| 401 | `'#/components/responses/Unauthorized'` | **All endpoints** |
| 403 | `'#/components/responses/Forbidden'` | **All endpoints** (RBAC enforced) |
| 404 | `'#/components/responses/NotFound'` | Endpoints that look up a resource by ID |
| 500 | `'#/components/responses/InternalError'` | **All endpoints** |

## Standard Rules

1. `summary` — ชื่อสั้น แสดงใน sidebar (ไม่เกิน 5-6 คำ)
2. `operationId` — camelCase, unique ทั้ง project (ใช้ใน SDK codegen)
3. `description` — เริ่มด้วย path `` `METHOD /api/path` `` แล้วอธิบาย
4. `example:` — วางก่อน `schema:` ใน content block (Scalar จะแสดง example นี้)
5. `required: [...]` — ระบุ field ที่การันตีว่ามีเสมอใน response object
6. YAML colon rule — ถ้า description มี `: ` ต้องใส่ `"..."` ครอบเสมอ
