# file-upload-tools

file-upload-tools is a powerful and flexible library designed to handle the splitting, processing, and uploading of large files in chunks.

## Table of Contents

- [Installation](#installation)
- [API Reference](#api-reference)
  - [currentFileChunks](#currentfilechunks)
  - [generateFileHash](#generatefilehash)
  - [PromisePool](#promisepool)
  - [uploadChunksWithPool](#uploadchunkswithpool)
- [Examples](#examples)
  - [Example: Splitting and Uploading a File](#example-splitting-and-uploading-a-file)
  - [Example: Generating a UUID](#example-generating-a-uuid)
  - [Example: Generating a File Hash in Chunks](#example-generating-a-file-hash-in-chunks)
  - [Example: Managing Concurrent Uploads with PromisePool](#example-managing-concurrent-uploads-with-promisepool)
  - [Example: Generating a File Hash with ArrayBuffer](#example-generating-a-file-hash-with-arraybuffer)
- [License](#license)

## Installation

To install the FileChunksTools library, use the following command:

```bash
npm install file-upload-tools
```

## API Reference

### `currentFileChunks`

Splits the given file into multiple chunks of the specified size.

**Parameters**:

- `file: File` - The file to be split.
- `customChunkSize?: number` - Custom size of each chunk (in MB). If not provided, the function automatically determines the chunk size.

**Returns**:

- `Promise<FileChunkResult>` - An object containing the file chunks and chunk size.

### `generateFileHash`

Calculate the hash of the given file using MD5.

**Parameters**:

- `file: File`
- `customChunkSize?: number`

**Returns**:

- `Promise<{ hash: string, chunkSize: number }>` - A promise that resolves to an object containing the hash and chunk size.

### `PromisePool`

Controls the execution of asynchronous tasks with a specified level of concurrency.

**Constructor Parameters**:

- `functions: AsyncFunction[]` - Array of task functions to be executed.
- `maxConcurrentTasks: number` - Maximum number of concurrent tasks.

**Methods**:

- `exec<T>(): Promise<T[]>` - Executes all asynchronous functions in the task pool.
- `pause(): void` - Pauses task execution.
- `resume(): void` - Resumes task execution.
- `clear(): void` - Clears the task queue.
- `addTasks(newTasks: AsyncFunction[]): void` - Adds new tasks to the queue.

### `uploadChunksWithPool`

Controls the concurrent upload of file chunks using PromisePool.

**Parameters**:

- `options: UploadOptions` - Configuration options for file upload.
- `cb: UploadCallback` - Callback function to handle the upload of a single file chunk.

**Returns**:

- `PromisePool` - Returns the instance of PromisePool managing the upload tasks.

## Examples

### Example: Splitting a File into Chunks

This example demonstrates how to use the `currentFileChunks` function to split a file into multiple chunks:

```typescript
import { currentFileChunks } from 'file-upload-tools';

async function splitFile(file: File) {
  // Split the file into chunks
  const { fileChunks, chunkSize } = await currentFileChunks(file);

  console.log('File has been split into the following chunks:');
  fileChunks.forEach((chunk, index) => {
    console.log(`Chunk ${index + 1} of size ${chunk.size} bytes`);
  });

  console.log('Chunk Size:', chunkSize);
}

// Example usage with a file input
const fileInput = document.querySelector('input[type="file"]');
fileInput.addEventListener('change', event => {
  const file = (event.target as HTMLInputElement).files[0];
  if (file) {
    splitFile(file);
  }
});
```

### Example: Generating a File Hash in Chunks

This example shows how to generate a hash for a large file using the `generateFileHash` function, which processes the file in chunks:

```typescript
import { generateFileHash } from 'file-upload-tools';

async function hashLargeFile(file: File) {
  const { hash, chunkSize } = await generateFileHash(file);

  console.log('Generated hash for the large file:', hash);
  console.log('Chunk Size:', chunkSize);
}

// Example usage with a file input
const fileInput = document.querySelector('input[type="file"]');
fileInput.addEventListener('change', event => {
  const file = (event.target as HTMLInputElement).files[0];
  if (file) {
    hashLargeFile(file);
  }
});
```

### Example: Managing Concurrent Uploads with PromisePool

This example demonstrates how to use the `uploadChunksWithPool` function to manage the concurrent upload of file chunks:

```typescript
import { currentFileChunks, uploadChunksWithPool } from 'file-upload-tools';

async function uploadFile(file: File) {
  // Step 1: Split the file into chunks
  const { fileChunks, chunkSize } = await currentFileChunks(file);

  // Step 2: Define the upload callback function
  const uploadChunk = async (chunk: Blob, index: number) => {
    console.log(`Uploading chunk ${index + 1} of ${fileChunks.length}`);
    // Simulate an upload request
    await fetch('/upload', {
      method: 'POST',
      body: chunk,
    });
  };

  // Step 3: Upload the chunks concurrently using PromisePool
  const pool = uploadChunksWithPool({ fileChunks, maxTasks: 4 }, uploadChunk);

  // Execute the upload tasks
  pool.exec().then(() => {
    console.log('All chunks uploaded successfully!');
  });
}

// Example usage with a file input
const fileInput = document.querySelector('input[type="file"]');
fileInput.addEventListener('change', event => {
  const file = (event.target as HTMLInputElement).files[0];
  if (file) {
    uploadFile(file);
  }
});
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
