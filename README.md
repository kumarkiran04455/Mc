# Mc
Better 
1.npm create vite@latest camonk-interview -- --template react-ts
cd camonk-interview
npm install
2.npm install @tanstack/react-query axios react-router-dom
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
3.npx shadcn@latest init
4.npx shadcn@latest add card button input textarea badge skeleton
5.import type { Config } from "tailwindcss"

const config: Config = {
  content: ["./index.html", "./src/**/*.{ts,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
}
export default config
6.@tailwind base;
@tailwind components;
@tailwind utilities;
7.{
  "blogs": [
    {
      "id": 1,
      "title": "Future of Fintech",
      "category": ["FINANCE", "TECH"],
      "description": "Exploring how AI and blockchain are reshaping financial services",
      "date": "2026-01-11T09:12:45.120Z",
      "coverImage": "https://images.pexels.com/photos/6801648/pexels-photo-6801648.jpeg",
      "content": "Full blog content..."
    }
  ]
}
8."scripts": {
  "dev": "vite",
  "server": "json-server --watch db.json --port 3001"
}
9.npm run server
10.src/
│── api/
│   └── blogs.ts
│── components/
│   ├── BlogCard.tsx
│   ├── BlogList.tsx
│   ├── BlogDetail.tsx
│   └── CreateBlog.tsx
│── pages/
│   └── Home.tsx
│── types/
│   └── blog.ts
│── App.tsx
│── main.tsx
11.export interface Blog {
  id: number
  title: string
  category: string[]
  description: string
  date: string
  coverImage: string
  content: string
}
12.import axios from "axios"
import { Blog } from "@/types/blog"

const API_URL = "http://localhost:3001/blogs"

export const getBlogs = async (): Promise<Blog[]> => {
  const res = await axios.get(API_URL)
  return res.data
}

export const getBlogById = async (id: number): Promise<Blog> => {
  const res = await axios.get(`${API_URL}/${id}`)
  return res.data
}

export const createBlog = async (blog: Omit<Blog, "id">) => {
  const res = await axios.post(API_URL, blog)
  return res.data
}
13.import React from "react"
import ReactDOM from "react-dom/client"
import App from "./App"
import "./index.css"

import { QueryClient, QueryClientProvider } from "@tanstack/react-query"
import { BrowserRouter } from "react-router-dom"

const queryClient = new QueryClient()

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        <App />
      </BrowserRouter>
    </QueryClientProvider>
  </React.StrictMode>
)
13.import { Card, CardContent } from "@/components/ui/card"
import { Blog } from "@/types/blog"

interface Props {
  blog: Blog
  onClick: () => void
}

export default function BlogCard({ blog, onClick }: Props) {
  return (
    <Card onClick={onClick} className="cursor-pointer hover:shadow-lg">
      <CardContent className="p-4 space-y-2">
        <div className="flex gap-2">
          {blog.category.map(cat => (
            <span key={cat} className="text-xs bg-gray-200 px-2 py-1 rounded">
              {cat}
            </span>
          ))}
        </div>
        <h3 className="font-bold">{blog.title}</h3>
        <p className="text-sm text-gray-600">{blog.description}</p>
      </CardContent>
    </Card>
  )
}
,
import { useQuery } from "@tanstack/react-query"
import { getBlogs } from "@/api/blogs"
import BlogCard from "./BlogCard"
import { Skeleton } from "@/components/ui/skeleton"

export default function BlogList({ onSelect }: { onSelect: (id: number) => void }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ["blogs"],
    queryFn: getBlogs,
  })

  if (isLoading) return <Skeleton className="h-40 w-full" />
  if (error) return <p>Error loading blogs</p>

  return (
    <div className="space-y-4">
      {data?.map(blog => (
        <BlogCard key={blog.id} blog={blog} onClick={() => onSelect(blog.id)} />
      ))}
    </div>
  )
}
import { useQuery } from "@tanstack/react-query"
import { getBlogById } from "@/api/blogs"

export default function BlogDetail({ blogId }: { blogId: number }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ["blog", blogId],
    queryFn: () => getBlogById(blogId),
    enabled: !!blogId,
  })

  if (!blogId) return <p>Select a blog</p>
  if (isLoading) return <p>Loading...</p>
  if (error) return <p>Error loading blog</p>

  return (
    <div className="space-y-4">
      <img src={data?.coverImage} className="rounded-lg" />
      <h1 className="text-2xl font-bold">{data?.title}</h1>
      <p>{data?.content}</p>
    </div>
  )
}
import { useMutation, useQueryClient } from "@tanstack/react-query"
import { createBlog } from "@/api/blogs"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Textarea } from "@/components/ui/textarea"
import { useState } from "react"

export default function CreateBlog() {
  const queryClient = useQueryClient()

  const [title, setTitle] = useState("")
  const [description, setDescription] = useState("")
  const [content, setContent] = useState("")

  const mutation = useMutation({
    mutationFn: createBlog,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["blogs"] })
    },
  })

  const submit = () => {
    mutation.mutate({
      title,
      description,
      content,
      category: ["GENERAL"],
      coverImage: "",
      date: new Date().toISOString(),
    })
  }

  return (
    <div className="space-y-3">
      <Input placeholder="Title" onChange={e => setTitle(e.target.value)} />
      <Input placeholder="Description" onChange={e => setDescription(e.target.value)} />
      <Textarea placeholder="Content" onChange={e => setContent(e.target.value)} />
      <Button onClick={submit}>Create Blog</Button>
    </div>
  )
}
import { useState } from "react"
import BlogList from "@/components/BlogList"
import BlogDetail from "@/components/BlogDetail"
import CreateBlog from "@/components/CreateBlog"

export default function Home() {
  const [selectedId, setSelectedId] = useState<number>(0)

  return (
    <div className="grid grid-cols-3 gap-6 p-6">
      <div className="col-span-1">
        <CreateBlog />
        <BlogList onSelect={setSelectedId} />
      </div>
      <div className="col-span-2">
        <BlogDetail blogId={selectedId} />
      </div>
    </div>
  )
}import Home from "./pages/Home"

export default function App() {
  return <Home />
}
