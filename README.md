# Vibeforge-
The vibeforge mega-App(Music, Art, Stories, AI math, Market place, fashion Generator, etc)
#!/usr/bin/env bash
set -euo pipefail

# Create project
PROJECT_DIR="vibeforge"
if [ -d "$PROJECT_DIR" ]; then
  echo "Directory $PROJECT_DIR already exists — skipping create-next-app step."
else
  npx create-next-app@latest "$PROJECT_DIR" --ts=false --eslint=false --app=false --use-npm --yes
fi

cd "$PROJECT_DIR"

# Install dependencies
npm install next@14 react@18 react-dom@18
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# Tailwind config (content + theme colors)
cat > tailwind.config.js <<'EOF'
module.exports = {
  content: ['./pages/**/*.{js,jsx}','./components/**/*.{js,jsx}'],
  theme: {
    extend: {
      colors: {
        'vibeforge-gold': '#D4AF37',
        'vibeforge-black': '#0b0b0b',
        'vibeforge-red': '#e03b3b',
        'vibeforge-green': '#2fa44f'
      }
    }
  },
  plugins: [],
}
EOF

# Global CSS
mkdir -p styles
cat > styles/globals.css <<'EOF'
@tailwind base;
@tailwind components;
@tailwind utilities;

:root{
  --gold:#D4AF37;
  --black:#0b0b0b;
  --accent:#e03b3b;
}

body{ background: linear-gradient(180deg,#0b0b0b 0%, #071018 100%); color:#fff; }

.card{ background: rgba(255,255,255,0.03); border:1px solid rgba(212,175,55,0.08); border-radius:12px; padding:16px; }
.logo{ font-weight:800; color:var(--gold); }
textarea, input { background: rgba(255,255,255,0.02); border:1px solid rgba(255,255,255,0.03); color:inherit; padding:8px; border-radius:6px; width:100%; }
button { cursor:pointer; }
EOF

# Create components folder and Layout component
mkdir -p components
cat > components/Layout.js <<'EOF'
import Head from 'next/head'
import Link from 'next/link'
export default function Layout({children}){
  return (
    <div>
      <Head>
        <title>VibeForge</title>
        <meta name="viewport" content="width=device-width, initial-scale=1" />
      </Head>
      <header className="p-4 flex items-center justify-between">
        <div className="flex items-center gap-4">
          <div className="logo text-2xl">VibeForge</div>
          <nav className="hidden md:flex gap-4 text-sm text-gray-300">
            <Link href="/music">Music</Link>
            <Link href="/art">Art</Link>
            <Link href="/math">Math</Link>
            <Link href="/poem">Writing</Link>
          </nav>
        </div>
        <div className="flex items-center gap-2">
          <Link href="/login"><button className="px-3 py-1 rounded bg-vibeforge-gold text-black">Login</button></Link>
        </div>
      </header>
      <main className="p-6 md:p-12">{children}</main>
      <footer className="p-6 text-center text-gray-400">© VibeForge — African Vibes • Built with ❤️</footer>
    </div>
  )
}
EOF

# Pages
mkdir -p pages/api/music
cat > pages/index.js <<'EOF'
import Link from 'next/link'
export default function Home(){
  return (
    <div className="max-w-6xl mx-auto">
      <section className="grid md:grid-cols-3 gap-6">
        <div className="card p-6">
          <h2 className="text-2xl font-bold">Music Studio</h2>
          <p>Create beats, melodies & full songs across genres.</p>
          <Link href="/music"><a className="mt-4 inline-block px-4 py-2 bg-vibeforge-gold text-black rounded">Open</a></Link>
        </div>
        <div className="card p-6">
          <h2 className="text-2xl font-bold">Art & Cartoon</h2>
          <p>Generate album covers, cartoons, and logos.</p>
          <Link href="/art"><a className="mt-4 inline-block px-4 py-2 bg-vibeforge-gold text-black rounded">Open</a></Link>
        </div>
        <div className="card p-6">
          <h2 className="text-2xl font-bold">Math Hub</h2>
          <p>ECZ syllabus, lessons, and university help.</p>
          <Link href="/math"><a className="mt-4 inline-block px-4 py-2 bg-vibeforge-gold text-black rounded">Open</a></Link>
        </div>
      </section>
    </div>
  )
}
EOF

cat > pages/music.js <<'EOF'
import {useState} from 'react'
export default function Music(){
  const [lyrics,setLyrics]=useState('I am dancing on the beat of the night')
  const [url,setUrl]=useState('')
  async function gen(){
    const r = await fetch('/api/music/lyrics2melody',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({lyrics})})
    const j = await r.json();
    const r2 = await fetch('/api/music/render',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({notes:j.notes||[], bpm:100})})
    if(r2.ok){ const blob = await r2.blob(); setUrl(URL.createObjectURL(blob)); }
  }
  return (<div className="max-w-3xl mx-auto card">
    <h1 className="text-2xl font-bold">Music Studio</h1>
    <textarea className="w-full mt-4 p-2" rows={6} value={lyrics} onChange={e=>setLyrics(e.target.value)} />
    <div className="mt-4"><button className="px-4 py-2 bg-vibeforge-gold text-black rounded" onClick={gen}>Generate</button></div>
    {url && <audio className="mt-4" controls src={url}></audio>}
  </div>)
}
EOF

cat > pages/art.js <<'EOF'
import {useState} from 'react'
export default function Art(){
  const [prompt,setPrompt]=useState('African sunset with drums')
  const [img,setImg]=useState('')
  async function gen(){ const r=await fetch('/api/art/generate',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({prompt})}); const j=await r.json(); setImg(j.url); }
  return (<div className="max-w-3xl mx-auto card">
    <h1 className="text-2xl font-bold">Art & Cartoon</h1>
    <textarea className="w-full mt-4 p-2" rows={4} value={prompt} onChange={e=>setPrompt(e.target.value)} />
    <div className="mt-4"><button className="px-4 py-2 bg-vibeforge-gold text-black rounded" onClick={gen}>Generate Art</button></div>
    {img && <img className="mt-4 w-full" src={img} alt='generated' />}
  </div>)
}
EOF

cat > pages/math.js <<'EOF'
import {useState} from 'react'
export default function Math(){
  const [prob,setProb]=useState('integrate x**2 dx')
  const [res,setRes]=useState(null)
  async function solve(){ const r=await fetch('/api/services/sympy/solve',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({problem:prob})}); const j=await r.json(); setRes(j) }
  return (<div className="max-w-3xl mx-auto card"><h1 className="text-2xl font-bold">Math Hub</h1>
    <textarea className="w-full mt-4 p-2" rows={4} value={prob} onChange={e=>setProb(e.target.value)} />
    <div className="mt-4"><button className="px-4 py-2 bg-vibeforge-gold text-black rounded" onClick={solve}>Solve</button></div>
    {res && <pre className="mt-4 bg-black/20 p-4 rounded">{JSON.stringify(res,null,2)}</pre>}
  </div>)
}
EOF

cat > pages/poem.js <<'EOF'
import {useState} from 'react'
export default function Poem(){
  const [theme,setTheme]=useState('Love and drums')
  const [poem,setPoem]=useState('')
  async function gen(){ const r=await fetch('/api/write/poem',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({theme})}); const t=await r.text(); setPoem(t) }
  return (<div className="max-w-3xl mx-auto card"><h1 className="text-2xl font-bold">Writing Studio</h1>
    <input className="w-full mt-4 p-2" value={theme} onChange={e=>setTheme(e.target.value)} />
    <div className="mt-4"><button className="px-4 py-2 bg-vibeforge-gold text-black rounded" onClick={gen}>Generate Poem</button></div>
    {poem && <pre className="mt-4 bg-black/20 p-4 rounded">{poem}</pre>}
  </div>)
}
EOF

cat > pages/login.js <<'EOF'
import {useState} from 'react'
import Router from 'next/router'
export default function Login(){
  const [email,setEmail]=useState('')
  const [password,setPassword]=useState('')
  async function submit(){ const r = await fetch('/api/auth/login',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({email,password})}); const j=await r.json(); if(j.token){ localStorage.setItem('vf_token', j.token); alert('Logged in'); Router.push('/'); } else alert('Login failed') }
  return (<div className="max-w-md mx-auto card"><h1 className="text-2xl font-bold">Login</h1>
    <input className="w-full mt-4 p-2" placeholder='email' value={email} onChange={e=>setEmail(e.target.value)} />
    <input className="w-full mt-4 p-2" placeholder='password' value={password} onChange={e=>setPassword(e.target.value)} type='password' />
    <div className="mt-4"><button className="px-4 py-2 bg-vibeforge-gold text-black rounded" onClick={submit}>Login</button></div>
  </div>)
}
EOF

# API stubs
cat > pages/api/art/generate.js <<'EOF'
export default async function handler(req,res){
  const { prompt } = req.body || {}
  return res.status(200).json({ url: `https://via.placeholder.com/1024x512.png?text=${encodeURIComponent(prompt||'vibeforge')}` })
}
EOF

cat > pages/api/music/lyrics2melody.js <<'EOF'
export default function handler(req,res){
  const { lyrics } = req.body || {}
  const words = (lyrics||'hello world').split(/\s+/).slice(0,16)
  const notes = words.map((w,i)=> ({note:60 + (i%12), time:i*0.5, dur:0.5}))
  res.json({notes, bpm:100})
}
EOF

cat > pages/api/music/render.js <<'EOF'
export default async function handler(req,res){
  const url = '/demo/vibeforge_demo_main.wav'
  res.redirect(url)
}
EOF

cat > pages/api/write/poem.js <<'EOF'
export default async function handler(req,res){
  const { theme } = req.body || {}
  return res.status(200).send(`A short poem about ${theme}:\n\nBeats like heart, drums like rain.`)
}
EOF

cat > pages/api/services/sympy/solve.js <<'EOF'
export default function handler(req,res){
  const { problem } = req.body || {}
  return res.json({ input: problem, solution: 'This is a placeholder solution. Replace with real SymPy backend.' })
}
EOF

cat > pages/api/market/items.js <<'EOF'
export default function handler(req,res){
  return res.status(200).json([{id:1,title:'Guitar',price:'ZMW 12000'},{id:2,title:'Keyboard',price:'ZMW 22000'}])
}
EOF

cat > pages/api/market/create_payment.js <<'EOF'
export default async function handler(req,res){
  const { itemId, provider, email } = req.body || {}
  if(provider==='flutterwave') return res.json({link:'https://flutterwave.com/pay/demo'})
  if(provider==='stripe') return res.json({client_secret:'demo_client_secret'})
  return res.status(400).json({error:'unknown provider'})
}
EOF

cat > pages/api/auth/login.js <<'EOF'
export default async function handler(req,res){
  const { email, password } = req.body || {}
  return res.json({ token: 'demo-token', user:{email} })
}
EOF

# public demo wav (small base64 silence clip)
mkdir -p public/demo
cat > public/demo/vibeforge_demo_main.wav <<'B64'
UklGRhQAAABXQVZFZm10IBAAAAABAAEAESsAACJWAAACABAAZGF0YRAAAAAA
B64
# decode it to binary
base64 --decode public/demo/vibeforge_demo_main.wav > public/demo/vibeforge_demo_main.wav.bin || true
# If decode succeeded, move to final; else leave as-is. Try to detect.
if [ -s public/demo/vibeforge_demo_main.wav.bin ]; then
  mv public/demo/vibeforge_demo_main.wav.bin public/demo/vibeforge_demo_main.wav
else
  echo "Could not decode demo WAV; a placeholder file remains."
fi

# Update package.json scripts (if using next default)
cat > package.json <<'EOF'
{
  "name": "vibeforge",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev -p 3000",
    "build": "next build",
    "start": "next start -p 3000"
  },
  "dependencies": {
    "next": "14.1.0",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "swr": "^2.2.0"
  },
  "devDependencies": {
    "tailwindcss": "^3.5.0",
    "autoprefixer": "^10.4.12",
    "postcss": "^8.4.18"
  }
}
EOF

# Install deps
npm install

# Final note
echo "Setup complete. Run 'npm run dev' inside the project directory (it should already be the current dir)."
echo "If Codespaces asks to forward port 3000, allow it to preview the app."