# IPFS HTTP Client, Pinata & Web3.Storage Cheat Sheet

## IPFS HTTP Client

### Installation
```bash
npm install ipfs-http-client
# For Node.js environments
npm install node-fetch
```

### Basic Setup

#### Client Creation
```typescript
import { create } from 'ipfs-http-client'

// Local IPFS node
const client = create({ 
  host: 'localhost', 
  port: 5001, 
  protocol: 'http' 
})

// Infura IPFS
const client = create({
  host: 'ipfs.infura.io',
  port: 5001,
  protocol: 'https',
  headers: {
    authorization: `Basic ${Buffer.from(`${projectId}:${projectSecret}`).toString('base64')}`
  }
})

// Custom gateway
const client = create({
  url: 'https://your-ipfs-gateway.com',
  headers: {
    'Authorization': 'Bearer your-token'
  }
})
```

### File Operations

#### Adding Files
```typescript
// Add single file
const file = new File(['Hello World'], 'hello.txt', { type: 'text/plain' })
const result = await client.add(file)
console.log('IPFS Hash:', result.cid.toString())

// Add multiple files
const files = [
  { path: 'file1.txt', content: 'Content 1' },
  { path: 'file2.txt', content: 'Content 2' },
  { path: 'folder/file3.txt', content: 'Content 3' }
]

for await (const result of client.addAll(files)) {
  console.log('Added:', result.path, result.cid.toString())
}

// Add with options
const result = await client.add(file, {
  pin: true,
  wrapWithDirectory: true,
  progress: (prog) => console.log(`Progress: ${prog}`)
})

// Add JSON data
const jsonData = { name: 'John', age: 30 }
const result = await client.add(JSON.stringify(jsonData))
```

#### Retrieving Files
```typescript
// Get file content
const chunks = []
for await (const chunk of client.cat('QmHash...')) {
  chunks.push(chunk)
}
const content = Buffer.concat(chunks).toString()

// Get file as stream
const stream = client.cat('QmHash...')
for await (const chunk of stream) {
  console.log(chunk.toString())
}

// Get file info
const stat = await client.files.stat('/ipfs/QmHash...')
console.log('File size:', stat.size)

// List directory contents
for await (const file of client.ls('QmHash...')) {
  console.log('File:', file.name, file.cid.toString())
}
```

### Directory Operations

#### Creating Directories
```typescript
// Create directory structure
const files = [
  { path: 'images/photo1.jpg', content: imageBuffer1 },
  { path: 'images/photo2.jpg', content: imageBuffer2 },
  { path: 'metadata.json', content: JSON.stringify(metadata) }
]

const results = []
for await (const result of client.addAll(files, { wrapWithDirectory: true })) {
  results.push(result)
  if (!result.path) {
    console.log('Directory CID:', result.cid.toString())
  }
}
```

### Pinning Operations

#### Pin Management
```typescript
// Pin a hash
await client.pin.add('QmHash...')

// Pin with options
await client.pin.add('QmHash...', {
  recursive: true,
  metadata: { name: 'Important File' }
})

// List pins
for await (const pin of client.pin.ls()) {
  console.log('Pinned:', pin.cid.toString(), pin.type)
}

// Remove pin
await client.pin.rm('QmHash...')

// Check if pinned
const isPinned = await client.pin.ls({ paths: 'QmHash...' })
```

### React Integration

#### File Upload Hook
```typescript
// hooks/useIPFS.ts
import { useState } from 'react'
import { create } from 'ipfs-http-client'

export function useIPFS() {
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const client = create({
    host: 'ipfs.infura.io',
    port: 5001,
    protocol: 'https',
    headers: {
      authorization: `Basic ${Buffer.from(`${process.env.NEXT_PUBLIC_INFURA_PROJECT_ID}:${process.env.NEXT_PUBLIC_INFURA_SECRET}`).toString('base64')}`
    }
  })

  const uploadFile = async (file: File) => {
    setLoading(true)
    setError(null)
    
    try {
      const result = await client.add(file, {
        pin: true,
        progress: (prog) => console.log(`Upload progress: ${prog}`)
      })
      
      return result.cid.toString()
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Upload failed')
      throw err
    } finally {
      setLoading(false)
    }
  }

  const uploadJSON = async (data: any) => {
    return uploadFile(new File([JSON.stringify(data)], 'metadata.json', { type: 'application/json' }))
  }

  return { uploadFile, uploadJSON, loading, error }
}
```

#### File Upload Component
```typescript
// components/FileUpload.tsx
import { useState } from 'react'
import { useIPFS } from '../hooks/useIPFS'

export function FileUpload() {
  const [selectedFile, setSelectedFile] = useState<File | null>(null)
  const [ipfsHash, setIpfsHash] = useState<string>('')
  const { uploadFile, loading, error } = useIPFS()

  const handleUpload = async () => {
    if (!selectedFile) return

    try {
      const hash = await uploadFile(selectedFile)
      setIpfsHash(hash)
    } catch (err) {
      console.error('Upload failed:', err)
    }
  }

  return (
    <div className="space-y-4">
      <input
        type="file"
        onChange={(e) => setSelectedFile(e.target.files?.[0] || null)}
        className="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded file:border-0 file:bg-blue-50 file:text-blue-700"
      />
      
      <button
        onClick={handleUpload}
        disabled={!selectedFile || loading}
        className="px-4 py-2 bg-blue-500 text-white rounded disabled:opacity-50"
      >
        {loading ? 'Uploading...' : 'Upload to IPFS'}
      </button>

      {error && (
        <div className="text-red-500">Error: {error}</div>
      )}

      {ipfsHash && (
        <div className="space-y-2">
          <div>IPFS Hash: {ipfsHash}</div>
          <a
            href={`https://ipfs.io/ipfs/${ipfsHash}`}
            target="_blank"
            rel="noopener noreferrer"
            className="text-blue-500 underline"
          >
            View on IPFS
          </a>
        </div>
      )}
    </div>
  )
}
```

---

## Pinata

### Installation
```bash
npm install @pinata/sdk
# For browser usage
npm install axios
```

### Setup

#### API Configuration
```typescript
import pinataSDK from '@pinata/sdk'

// Initialize with API keys
const pinata = new pinataSDK({
  pinataApiKey: process.env.PINATA_API_KEY,
  pinataSecretApiKey: process.env.PINATA_SECRET_API_KEY
})

// Test authentication
const testAuth = async () => {
  try {
    const result = await pinata.testAuthentication()
    console.log('Pinata authenticated:', result)
  } catch (error) {
    console.error('Authentication failed:', error)
  }
}
```

### File Operations

#### Pin File from Filesystem (Node.js)
```typescript
import fs from 'fs'
import path from 'path'

// Pin single file
const pinFileToIPFS = async (filePath: string) => {
  const readableStreamForFile = fs.createReadStream(filePath)
  
  const options = {
    pinataMetadata: {
      name: path.basename(filePath),
      keyvalues: {
        customKey: 'customValue',
        type: 'image'
      }
    },
    pinataOptions: {
      cidVersion: 0
    }
  }

  try {
    const result = await pinata.pinFileToIPFS(readableStreamForFile, options)
    console.log('File pinned:', result.IpfsHash)
    return result
  } catch (error) {
    console.error('Error pinning file:', error)
    throw error
  }
}
```

#### Pin File from Buffer
```typescript
// Pin from buffer
const pinBufferToIPFS = async (buffer: Buffer, fileName: string) => {
  const options = {
    pinataMetadata: {
      name: fileName,
    },
    pinataOptions: {
      cidVersion: 1
    }
  }

  try {
    const result = await pinata.pinFileToIPFS(buffer, options)
    return result.IpfsHash
  } catch (error) {
    console.error('Error pinning buffer:', error)
    throw error
  }
}
```

#### Pin JSON Data
```typescript
// Pin JSON object
const pinJSONToIPFS = async (jsonData: any, name?: string) => {
  const options = {
    pinataMetadata: {
      name: name || 'JSON Data',
      keyvalues: {
        type: 'json',
        timestamp: new Date().toISOString()
      }
    }
  }

  try {
    const result = await pinata.pinJSONToIPFS(jsonData, options)
    console.log('JSON pinned:', result.IpfsHash)
    return result
  } catch (error) {
    console.error('Error pinning JSON:', error)
    throw error
  }
}

// Example usage
const metadata = {
  name: 'My NFT',
  description: 'A unique NFT',
  image: 'ipfs://QmHash...',
  attributes: [
    { trait_type: 'Color', value: 'Blue' },
    { trait_type: 'Rarity', value: 'Rare' }
  ]
}

const result = await pinJSONToIPFS(metadata, 'NFT Metadata')
```

### Pin Management

#### List Pins
```typescript
// Get all pins
const listPins = async () => {
  const filters = {
    status: 'pinned',
    pageLimit: 10,
    pageOffset: 0,
    metadata: {
      keyvalues: {
        type: {
          value: 'image',
          op: 'eq'
        }
      }
    }
  }

  try {
    const result = await pinata.pinList(filters)
    console.log('Total pins:', result.count)
    
    result.rows.forEach(pin => {
      console.log('Pin:', {
        hash: pin.ipfs_pin_hash,
        name: pin.metadata.name,
        size: pin.size,
        date: pin.date_pinned
      })
    })
    
    return result
  } catch (error) {
    console.error('Error listing pins:', error)
    throw error
  }
}
```

#### Unpin Files
```typescript
// Unpin file
const unpinFile = async (hashToUnpin: string) => {
  try {
    const result = await pinata.unpin(hashToUnpin)
    console.log('File unpinned:', result)
    return result
  } catch (error) {
    console.error('Error unpinning file:', error)
    throw error
  }
}
```

#### Update Pin Metadata
```typescript
// Update metadata
const updatePinMetadata = async (hashToUpdate: string, newMetadata: any) => {
  const metadata = {
    name: newMetadata.name,
    keyvalues: newMetadata.keyvalues
  }

  try {
    const result = await pinata.hashMetadata(hashToUpdate, metadata)
    console.log('Metadata updated:', result)
    return result
  } catch (error) {
    console.error('Error updating metadata:', error)
    throw error
  }
}
```

### Next.js API Routes

#### Upload Endpoint
```typescript
// pages/api/upload-to-pinata.ts
import type { NextApiRequest, NextApiResponse } from 'next'
import pinataSDK from '@pinata/sdk'
import formidable from 'formidable'
import fs from 'fs'

export const config = {
  api: {
    bodyParser: false,
  },
}

const pinata = new pinataSDK({
  pinataApiKey: process.env.PINATA_API_KEY!,
  pinataSecretApiKey: process.env.PINATA_SECRET_API_KEY!
})

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' })
  }

  try {
    const form = formidable({})
    const [fields, files] = await form.parse(req)
    
    const file = Array.isArray(files.file) ? files.file[0] : files.file
    if (!file) {
      return res.status(400).json({ error: 'No file uploaded' })
    }

    const stream = fs.createReadStream(file.filepath)
    
    const result = await pinata.pinFileToIPFS(stream, {
      pinataMetadata: {
        name: file.originalFilename || 'Uploaded File'
      }
    })

    res.status(200).json({
      success: true,
      ipfsHash: result.IpfsHash,
      pinSize: result.PinSize
    })
  } catch (error) {
    console.error('Upload error:', error)
    res.status(500).json({ error: 'Upload failed' })
  }
}
```

#### JSON Upload Endpoint
```typescript
// pages/api/pin-json.ts
import type { NextApiRequest, NextApiResponse } from 'next'
import pinataSDK from '@pinata/sdk'

const pinata = new pinataSDK({
  pinataApiKey: process.env.PINATA_API_KEY!,
  pinataSecretApiKey: process.env.PINATA_SECRET_API_KEY!
})

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' })
  }

  try {
    const { data, name } = req.body

    const result = await pinata.pinJSONToIPFS(data, {
      pinataMetadata: {
        name: name || 'JSON Data'
      }
    })

    res.status(200).json({
      success: true,
      ipfsHash: result.IpfsHash
    })
  } catch (error) {
    console.error('JSON pin error:', error)
    res.status(500).json({ error: 'Failed to pin JSON' })
  }
}
```

### React Hooks

#### Pinata Upload Hook
```typescript
// hooks/usePinata.ts
import { useState } from 'react'

interface UsePinataReturn {
  uploadFile: (file: File) => Promise<string>
  uploadJSON: (data: any, name?: string) => Promise<string>
  loading: boolean
  error: string | null
}

export function usePinata(): UsePinataReturn {
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const uploadFile = async (file: File): Promise<string> => {
    setLoading(true)
    setError(null)

    try {
      const formData = new FormData()
      formData.append('file', file)

      const response = await fetch('/api/upload-to-pinata', {
        method: 'POST',
        body: formData
      })

      const result = await response.json()
      
      if (!result.success) {
        throw new Error(result.error)
      }

      return result.ipfsHash
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Upload failed'
      setError(errorMessage)
      throw new Error(errorMessage)
    } finally {
      setLoading(false)
    }
  }

  const uploadJSON = async (data: any, name?: string): Promise<string> => {
    setLoading(true)
    setError(null)

    try {
      const response = await fetch('/api/pin-json', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ data, name })
      })

      const result = await response.json()
      
      if (!result.success) {
        throw new Error(result.error)
      }

      return result.ipfsHash
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'JSON upload failed'
      setError(errorMessage)
      throw new Error(errorMessage)
    } finally {
      setLoading(false)
    }
  }

  return { uploadFile, uploadJSON, loading, error }
}
```

---

## Web3.Storage

### Installation
```bash
npm install web3.storage
```

### Setup

#### Client Configuration
```typescript
import { Web3Storage } from 'web3.storage'

// Initialize client
const client = new Web3Storage({ token: process.env.WEB3_STORAGE_TOKEN! })

// Check client info
const info = await client.status()
console.log('Client info:', info)
```

### File Operations

#### Upload Files
```typescript
// Upload single file
const uploadFile = async (file: File) => {
  try {
    const cid = await client.put([file], {
      name: 'My Upload',
      maxRetries: 3,
      onRootCidReady: (rootCid) => {
        console.log('Root CID ready:', rootCid)
      },
      onStoredChunk: (size) => {
        console.log('Stored chunk of size:', size)
      }
    })
    
    console.log('Upload complete! CID:', cid)
    return cid
  } catch (error) {
    console.error('Upload failed:', error)
    throw error
  }
}

// Upload multiple files
const uploadMultipleFiles = async (files: File[]) => {
  const cid = await client.put(files, {
    name: 'Multiple Files Upload',
    wrapWithDirectory: true
  })
  
  return cid
}

// Upload from buffer
const uploadBuffer = async (buffer: ArrayBuffer, fileName: string) => {
  const file = new File([buffer], fileName)
  return await client.put([file])
}
```

#### Retrieve Files
```typescript
// Get file by CID
const getFile = async (cid: string, fileName?: string) => {
  try {
    const res = await client.get(cid)
    
    if (!res.ok) {
      throw new Error(`Failed to get ${cid}`)
    }
    
    const files = await res.files()
    
    if (fileName) {
      return files.find(f => f.name === fileName)
    }
    
    return files[0] // Return first file
  } catch (error) {
    console.error('Get file failed:', error)
    throw error
  }
}

// Get all files in a directory
const getDirectory = async (cid: string) => {
  const res = await client.get(cid)
  const files = await res.files()
  
  return files.map(file => ({
    name: file.name,
    size: file.size,
    content: file // Use file.stream() to get content
  }))
}
```

### Advanced Operations

#### List Uploads
```typescript
// List all uploads
const listUploads = async () => {
  const uploads = []
  
  for await (const upload of client.list()) {
    uploads.push({
      cid: upload.cid,
      name: upload.name,
      created: upload.created,
      dagSize: upload.dagSize,
      pins: upload.pins
    })
  }
  
  return uploads
}

// List with filtering
const listRecentUploads = async (limit = 10) => {
  const uploads = []
  let count = 0
  
  for await (const upload of client.list({ maxResults: limit })) {
    uploads.push(upload)
    count++
    if (count >= limit) break
  }
  
  return uploads
}
```

#### Check Upload Status
```typescript
// Get upload status
const getUploadStatus = async (cid: string) => {
  try {
    const status = await client.status(cid)
    
    return {
      cid: status.cid,
      name: status.name,
      created: status.created,
      dagSize: status.dagSize,
      pins: status.pins.map(pin => ({
        location: pin.location,
        status: pin.status
      }))
    }
  } catch (error) {
    console.error('Status check failed:', error)
    throw error
  }
}
```

#### Delete Uploads
```typescript
// Delete upload
const deleteUpload = async (cid: string) => {
  try {
    await client.delete(cid)
    console.log('Upload deleted:', cid)
  } catch (error) {
    console.error('Delete failed:', error)
    throw error
  }
}
```

### React Integration

#### Web3.Storage Hook
```typescript
// hooks/useWeb3Storage.ts
import { useState } from 'react'
import { Web3Storage } from 'web3.storage'

export function useWeb3Storage() {
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)
  
  const client = new Web3Storage({ 
    token: process.env.NEXT_PUBLIC_WEB3_STORAGE_TOKEN! 
  })

  const uploadFiles = async (files: File[], options?: { name?: string }) => {
    setLoading(true)
    setError(null)
    
    try {
      const cid = await client.put(files, {
        name: options?.name || 'Upload',
        onRootCidReady: (rootCid) => {
          console.log('Root CID:', rootCid)
        },
        onStoredChunk: (size) => {
          console.log('Stored:', size)
        }
      })
      
      return cid
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Upload failed'
      setError(errorMessage)
      throw new Error(errorMessage)
    } finally {
      setLoading(false)
    }
  }

  const getFile = async (cid: string, fileName?: string) => {
    setLoading(true)
    setError(null)
    
    try {
      const res = await client.get(cid)
      if (!res.ok) throw new Error('Failed to fetch')
      
      const files = await res.files()
      return fileName ? files.find(f => f.name === fileName) : files[0]
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Fetch failed'
      setError(errorMessage)
      throw new Error(errorMessage)
    } finally {
      setLoading(false)
    }
  }

  const listUploads = async () => {
    setLoading(true)
    setError(null)
    
    try {
      const uploads = []
      for await (const upload of client.list()) {
        uploads.push(upload)
      }
      return uploads
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'List failed'
      setError(errorMessage)
      throw new Error(errorMessage)
    } finally {
      setLoading(false)
    }
  }

  return {
    uploadFiles,
    getFile,
    listUploads,
    loading,
    error
  }
}
```

#### Upload Component
```typescript
// components/Web3StorageUpload.tsx
import { useState } from 'react'
import { useWeb3Storage } from '../hooks/useWeb3Storage'

export function Web3StorageUpload() {
  const [selectedFiles, setSelectedFiles] = useState<File[]>([])
  const [uploadName, setUploadName] = useState('')
  const [cid, setCid] = useState('')
  const { uploadFiles, loading, error } = useWeb3Storage()

  const handleUpload = async () => {
    if (selectedFiles.length === 0) return

    try {
      const resultCid = await uploadFiles(selectedFiles, { 
        name: uploadName || 'My Upload' 
      })
      setCid(resultCid)
    } catch (err) {
      console.error('Upload error:', err)
    }
  }

  return (
    <div className="space-y-4 p-4 border rounded">
      <div>
        <label className="block text-sm font-medium mb-2">
          Upload Name
        </label>
        <input
          type="text"
          value={uploadName}
          onChange={(e) => setUploadName(e.target.value)}
          placeholder="Optional upload name"
          className="w-full px-3 py-2 border rounded"
        />
      </div>

      <div>
        <label className="block text-sm font-medium mb-2">
          Select Files
        </label>
        <input
          type="file"
          multiple
          onChange={(e) => {
            const files = Array.from(e.target.files || [])
            setSelectedFiles(files)
          }}
          className="w-full"
        />
      </div>

      {selectedFiles.length > 0 && (
        <div>
          <p className="text-sm text-gray-600">
            Selected {selectedFiles.length} file(s):
          </p>
          <ul className="text-sm">
            {selectedFiles.map((file, index) => (
              <li key={index}>
                {file.name} ({(file.size / 1024).toFixed(2)} KB)
              </li>
            ))}
          </ul>
        </div>
      )}

      <button
        onClick={handleUpload}
        disabled={selectedFiles.length === 0 || loading}
        className="w-full px-4 py-2 bg-blue-500 text-white rounded disabled:opacity-50"
      >
        {loading ? 'Uploading...' : 'Upload to Web3.Storage'}
      </button>

      {error && (
        <div className="p-3 bg-red-100 border border-red-400 text-red-700 rounded">
          Error: {error}
        </div>
      )}

      {cid && (
        <div className="p-3 bg-green-100 border border-green-400 text-green-700 rounded">
          <p><strong>Upload successful!</strong></p>
          <p className="break-all">CID: {cid}</p>
          <a
            href={`https://${cid}.ipfs.w3s.link`}
            target="_blank"
            rel="noopener noreferrer"
            className="text-blue-600 underline"
          >
            View on IPFS Gateway
          </a>
        </div>
      )}
    </div>
  )
}
```

## Comparison: IPFS HTTP Client vs Pinata vs Web3.Storage

| Feature | IPFS HTTP Client | Pinata | Web3.Storage |
|---------|------------------|---------|--------------|
| **Setup Complexity** | Medium | Easy | Easy |
| **Pricing** | Free (own node) | Freemium | Free quota |
| **Reliability** | Depends on node | High | High |
| **API Quality** | Good | Excellent | Excellent |
| **Metadata Support** | Basic | Rich | Basic |
| **Browser Support** | Limited | Excellent | Excellent |
| **Performance** | Variable | High | High |

### When to Use Which

**IPFS HTTP Client:**
- You run your own IPFS node
- Need full control over IPFS operations
- Building infrastructure-level applications
- Want direct IPFS protocol access

**Pinata:**
- Need reliable pinning service
- Want rich metadata and organization features
- Building production applications
- Need analytics and management tools

**Web3.Storage:**
- Want free, reliable storage
- Building on Filecoin ecosystem
- Need simple, clean API
- Want automatic redundancy

## Best Practices

### File Organization
```typescript
// Organize files in meaningful structures
const uploadNFTCollection = async (images: File[], metadata: any[]) => {
  const files = [
    ...images.map((img, i) => ({ path: `images/${i}.jpg`, content: img })),
    ...metadata.map((meta, i) => ({ 
      path: `metadata/${i}.json`, 
      content: JSON.stringify(meta) 
    }))
  ]
  
  // This creates a directory structure on IPFS
  return await client.addAll(files, { wrapWithDirectory: true })
}
```

### Error Handling
```typescript
const robustUpload = async (file: File, maxRetries = 3) => {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await uploadToIPFS(file)
    } catch (error) {
      console.log(`Attempt ${attempt} failed:`, error)
      
      if (attempt === maxRetries) {
        throw new Error(`Upload failed after ${maxRetries} attempts`)
      }
      
      // Wait before retry
      await new Promise(resolve => setTimeout(resolve, 1000 * attempt))
    }
  }
}
```

### Environment Variables
```bash
# .env.local

# IPFS HTTP Client (Infura)
NEXT_PUBLIC_INFURA_PROJECT_ID=your_project_id
NEXT_PUBLIC_INFURA_SECRET=your_project_secret

# Pinata
PINATA_API_KEY=your_api_key
PINATA_SECRET_API_KEY=your_secret_key

# Web3.Storage
NEXT_PUBLIC_WEB3_STORAGE_TOKEN=your_token
```

### Security Considerations
- Never expose API keys in frontend code
- Use API routes for sensitive operations
- Validate file types and sizes
- Implement rate limiting
- Use HTTPS for all requests
- Consider content moderation for public uploads