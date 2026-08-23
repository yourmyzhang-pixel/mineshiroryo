import { useState, useEffect, useRef, useCallback } from "react";
import { createClient } from '@supabase/supabase-js';
import {
  Heart, MessageCircle, Plus, X, Lock, Unlock, Image as ImageIcon,
  Tag, Mail, Phone, Instagram, Send, ChevronLeft, ChevronRight, ChevronDown,
  Trash2, Pencil, Check, Home as HomeIcon, LayoutGrid, Images,
  CreditCard, Loader2, Sparkles, Palette, Facebook, Twitter, Globe, Link2
} from "lucide-react";

/* ---------- Supabase Setup ---------- */
const supabaseUrl = 'https://jorcrybqarhzsnbvuxni.supabase.co';
const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImpvcmNyeWJxYXJoenNuYnZ1eG5pIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODc0OTA1NjQsImV4cCI6MjEwMzA2NjU2NH0.V6XV19uXUxsmSq-OpFNa4ya33lj31CKTQ4JPkTp5cBY';
const supabase = createClient(supabaseUrl, supabaseKey);

/* ---------- fonts + tiny global styles ---------- */
const FontStyle = () => (
  <style>{`
    @import url('https://fonts.googleapis.com/css2?family=Prompt:wght@400;500;600;700&family=IBM+Plex+Sans+Thai:wght@300;400;500;600&display=swap');
    .font-display { font-family: 'Prompt', sans-serif; }
    .font-body { font-family: 'IBM Plex Sans Thai', sans-serif; }
    .blob { border-radius: 62% 38% 55% 45% / 48% 42% 58% 52%; }
    .dashed-divider { background-image: linear-gradient(to right, #B0B0B0 55%, transparent 0%); background-position: bottom; background-size: 12px 2px; background-repeat: repeat-x; height: 2px; }
    .scrollbar-thin::-webkit-scrollbar { height: 8px; width: 8px; }
    .scrollbar-thin::-webkit-scrollbar-thumb { background: #E0E0E0; border-radius: 999px; }
    .scrollbar-thin::-webkit-scrollbar-track { background: transparent; }
    @keyframes gentle-bounce { 0%,100%{ transform: translateY(0);} 50%{ transform: translateY(-6px);} }
    .gentle-bounce { animation: gentle-bounce 2.2s ease-in-out infinite; }
    @media (prefers-reduced-motion: reduce) { .gentle-bounce { animation: none; } }
    input:focus, textarea:focus, button:focus-visible { outline: 2px solid #262626; outline-offset: 2px; }
    .c-text-1A1A1A { color: #1A1A1A; } .c-border-E0E0E0 { border-color: #E0E0E0; } .c-bg-F2F2F2 { background-color: #F2F2F2; }
    .c-text-6B6B6B { color: #6B6B6B; } .c-border-1A1A1A { border-color: #1A1A1A; } .c-text-A3A3A3 { color: #A3A3A3; }
    .c-bg-FFFFFF { background-color: #FFFFFF; } .c-text-262626 { color: #262626; } .c-bg-FFFFFF-90 { background-color: rgba(255,255,255,0.9); }
    .c-border-EBEBEB { border-color: #EBEBEB; } .c-bg-1A1A1A { background-color: #1A1A1A; } .c-border-D0D0D0 { border-color: #D0D0D0; }
    .c-bg-E5E5E5-40 { background-color: rgba(229,229,229,0.4); } .c-text-4B4B4B { color: #4B4B4B; } .c-text-B0B0B0 { color: #B0B0B0; }
    .c-text-000000 { color: #000000; } .c-text-CACACA { color: #CACACA; }
    .cover-gradient { background-image: linear-gradient(to bottom, rgba(0,0,0,0.1), rgba(0,0,0,0.2), #FFFFFF); }
    .hoverc-bg-F2F2F2:hover { background-color: #F2F2F2; } .hoverc-text-1A1A1A:hover { color: #1A1A1A; }
    .group:hover .grouphoverc-text-000000 { color: #000000; }
  `}</style>
);

/* ---------- defaults ---------- */
const DEFAULT_CONTENT = {
  profile: { eyebrow: "Hey there ✿", title: "My Little Corner", subtitle: "Photos, stories, and little favorite things, all in one place", buttonText: "Check out the feed →" },
  posts: [], albums: [], tags: [], hiddenTags: [],
  contact: { avatar: null, name: "My Name", role: "Storyteller through photos", bio: "Stop by to say hi or chat about anything", email: "", phone: "", instagram: "", line: "", links: [], hiddenFields: [] },
  nav: { logo: null }, footer: "made with ✿ — a little site you can edit yourself",
  theme: { accent: "#FF6F91" },
};
const ACCENT_PRESETS = ["#FF6F91", "#F59E0B", "#22C55E", "#3B82F6", "#8B5CF6", "#EF4444", "#14B8A6", "#1A1A1A"];
const LINK_TYPES = [
  { key: "facebook", label: "Facebook", icon: Facebook }, { key: "twitter", label: "X", icon: Twitter },
  { key: "website", label: "Website", icon: Globe }, { key: "other", label: "Other", icon: Link2 },
];
const uid = () => Date.now().toString(36) + Math.random().toString(36).slice(2, 8);
const tagLabel = (t) => `#${t}`;
const TAG_EMOJIS = ["🏷️", "🌊", "🌸", "🍜", "🎉", "📸", "☕", "🎵", "💡", "❤️", "🌙", "🐾", "✈️", "🎨"];

/* ---------- image compression + Supabase Upload ---------- */
function compressImage(file, maxWidth = 1400, quality = 0.82) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = (e) => {
      const img = new Image();
      img.onload = () => {
        let { width, height } = img;
        if (width > maxWidth) { height = Math.round(height * (maxWidth / width)); width = maxWidth; }
        const canvas = document.createElement("canvas");
        canvas.width = width; canvas.height = height;
        const ctx = canvas.getContext("2d");
        ctx.drawImage(img, 0, 0, width, height);
        resolve(canvas.toDataURL("image/jpeg", quality));
      };
      img.onerror = reject; img.src = e.target.result;
    };
    reader.onerror = reject; reader.readAsDataURL(file);
  });
}

async function uploadToSupabase(file, maxWidth = 1400) {
  const dataUrl = await compressImage(file, maxWidth, 0.82);
  const res = await fetch(dataUrl);
  const blob = await res.blob();
  const fileName = `img_${Date.now()}_${Math.random().toString(36).slice(2, 8)}.jpg`;
  
  const { error } = await supabase.storage.from('images').upload(fileName, blob, { 
    contentType: 'image/jpeg', cacheControl: '3600', upsert: false 
  });
  if (error) throw error;
  
  const { data: publicUrlData } = supabase.storage.from('images').getPublicUrl(fileName);
  return publicUrlData.publicUrl;
}

/* ---------- UI Components ---------- */
const Chip = ({ active, onClick, children, accent = "#262626" }) => (
  <button onClick={onClick} style={active ? { backgroundColor: accent, borderColor: accent } : undefined}
    className={`px-3 py-1.5 rounded-full text-sm font-body border transition-colors ${active ? "text-white" : "bg-white/70 c-text-1A1A1A c-border-E0E0E0 hoverc-bg-F2F2F2"}`}>{children}</button>
);
const FieldLabel = ({ children }) => <label className="block text-xs font-body c-text-6B6B6B mb-1">{children}</label>;
const EmojiPicker = ({ value, onChange }) => (
  <div className="flex gap-1 overflow-x-auto pb-1 mb-1.5 scrollbar-thin">
    {TAG_EMOJIS.map((e) => (
      <button key={e} type="button" onClick={() => onChange(e)} title={e}
        className={`flex-shrink-0 w-7 h-7 rounded-full flex items-center justify-center text-sm border transition-colors ${value === e ? "c-border-1A1A1A c-bg-F2F2F2" : "border-transparent hoverc-bg-F2F2F2"}`}>{e}</button>
    ))}
  </div>
);

function ThemeColorPicker({ accent, onChange, onClose }) {
  return (
    <div className="absolute top-12 right-0 z-50 bg-white rounded-2xl shadow-xl border c-border-E0E0E0 p-4 w-64">
      <div className="flex items-center justify-between mb-3"><span className="text-sm font-display font-semibold c-text-1A1A1A">Accent color</span><button onClick={onClose} className="c-text-A3A3A3"><X size={15} /></button></div>
      <div className="grid grid-cols-4 gap-2 mb-3">
        {ACCENT_PRESETS.map((c) => (<button key={c} onClick={() => onChange(c)} className="w-10 h-10 rounded-full border-2 transition-transform hover:scale-110" style={{ backgroundColor: c, borderColor: c === accent ? "#1A1A1A" : "transparent" }}/>))}
      </div>
      <div className="flex items-center gap-2">
        <input type="color" value={accent} onChange={(e) => onChange(e.target.value)} className="w-9 h-9 rounded-lg border c-border-E0E0E0 cursor-pointer bg-transparent" />
        <span className="text-xs c-text-6B6B6B font-body">Custom color</span>
      </div>
    </div>
  );
}

function AgeGateModal({ onConfirm, accent }) {
  return (
    <div className="fixed inset-0 z-[100] flex items-center justify-center p-4" style={{ backgroundColor: "rgba(10,10,10,0.92)" }}>
      <div className="c-bg-FFFFFF rounded-3xl p-7 w-full max-w-sm shadow-2xl font-body text-center">
        <div className="text-3xl mb-2">🔞</div><h3 className="font-display font-semibold text-lg c-text-1A1A1A mb-2">Mature content ahead</h3>
        <p className="text-sm c-text-6B6B6B mb-6 leading-relaxed">This site may contain content intended for adults only (18+). By continuing, you confirm that you are at least 18 years old.</p>
        <div className="flex flex-col gap-2">
          <button onClick={onConfirm} style={{ backgroundColor: accent }} className="w-full py-2.5 rounded-xl text-white font-medium hover:opacity-90">I'm 18+ — Enter</button>
          <button onClick={() => window.location.href = "https://www.google.com"} className="w-full py-2.5 rounded-xl border c-border-E0E0E0 c-text-6B6B6B text-sm hoverc-bg-F2F2F2">Leave</button>
        </div>
      </div>
    </div>
  );
}

function OwnerModal({ onClose, onLogin, error, accent }) {
  const [pw, setPw] = useState("");
  return (
    <div className="fixed inset-0 z-50 backdrop-blur-sm flex items-center justify-center p-4" style={{ backgroundColor: "rgba(0,0,0,0.4)" }}>
      <div className="c-bg-FFFFFF rounded-3xl p-6 w-full max-w-sm shadow-xl relative font-body">
        <button onClick={onClose} className="absolute top-4 right-4 c-text-6B6B6B"><X size={18} /></button>
        <h3 className="font-display font-semibold text-lg c-text-1A1A1A mb-1">Sign in as owner</h3>
        <p className="text-sm c-text-6B6B6B mb-4">Enter the password to edit this site</p>
        <input type="password" value={pw} onChange={(e) => setPw(e.target.value)} onKeyDown={(e) => e.key === "Enter" && onLogin(pw)} className="w-full rounded-xl border c-border-E0E0E0 px-3 py-2 mb-3 bg-white" autoFocus />
        {error && <p className="text-xs text-rose-600 mb-2">{error}</p>}
        <button onClick={() => onLogin(pw)} style={{ backgroundColor: accent }} className="w-full mt-1 py-2.5 rounded-xl text-white font-medium hover:opacity-90">Sign in</button>
      </div>
    </div>
  );
}

function ConfirmModal({ message, onConfirm, onCancel, accent }) {
  return (
    <div className="fixed inset-0 z-50 backdrop-blur-sm flex items-center justify-center p-4" style={{ backgroundColor: "rgba(0,0,0,0.4)" }}>
      <div className="c-bg-FFFFFF rounded-3xl p-6 w-full max-w-sm shadow-xl font-body">
        <p className="text-sm c-text-1A1A1A mb-5 leading-relaxed">{message}</p>
        <div className="flex gap-2">
          <button onClick={onCancel} className="flex-1 py-2.5 rounded-xl border c-border-E0E0E0 c-text-6B6B6B text-sm hoverc-bg-F2F2F2">Cancel</button>
          <button onClick={onConfirm} style={{ backgroundColor: accent }} className="flex-1 py-2.5 rounded-xl text-white text-sm font-medium hover:opacity-90">Confirm</button>
        </div>
      </div>
    </div>
  );
}

function AddAlbumModal({ onClose, onSubmit, accent }) {
  const [name, setName] = useState("");
  const [nsfw, setNsfw] = useState(false);
  return (
    <div className="fixed inset-0 z-50 backdrop-blur-sm flex items-center justify-center p-4" style={{ backgroundColor: "rgba(0,0,0,0.5)" }}>
      <div className="c-bg-FFFFFF rounded-3xl p-6 w-full max-w-sm shadow-2xl relative font-body">
        <button onClick={onClose} className="absolute top-4 right-4 c-text-6B6B6B"><X size={18} /></button>
        <h3 className="font-display font-semibold text-lg c-text-1A1A1A mb-4">Create New Album</h3>
        <FieldLabel>Album name</FieldLabel>
        <input autoFocus value={name} onChange={(e) => setName(e.target.value)} placeholder="e.g. Beach Trip" className="w-full rounded-xl border c-border-E0E0E0 px-3 py-2 mb-3 bg-white text-sm" />
        <button type="button" onClick={() => setNsfw(!nsfw)} className={`w-full flex items-center justify-between gap-2 rounded-xl border px-3 py-2.5 mb-4 transition-colors ${nsfw ? "border-transparent" : "c-border-E0E0E0"}`} style={nsfw ? { backgroundColor: "#FFF1F2", borderColor: "#FDA4AF" } : undefined}>
          <span className="flex items-center gap-1.5 text-sm font-body c-text-1A1A1A">🔞 Mark album as NSFW</span>
          <span className={`relative w-9 h-5 rounded-full transition-colors flex-shrink-0 ${nsfw ? "" : "c-bg-F2F2F2"}`} style={nsfw ? { backgroundColor: accent } : undefined}><span className={`absolute top-0.5 w-4 h-4 rounded-full bg-white shadow transition-all ${nsfw ? "left-4" : "left-0.5"}`} /></span>
        </button>
        <button onClick={() => { if(name.trim()) onSubmit(name.trim(), nsfw) }} disabled={!name.trim()} style={{ backgroundColor: accent }} className="w-full py-2.5 rounded-xl text-white font-medium disabled:opacity-40 hover:opacity-90">Create Album</button>
      </div>
    </div>
  );
}

/* ================= APP MAIN ================= */
const OWNER_PASSWORD = "XENOMEGAX";

export default function App() {
  const [content, setContent] = useState(DEFAULT_CONTENT);
  const [loading, setLoading] = useState(true);
  const [activeTab, setActiveTab] = useState("home");
  const [isOwner, setIsOwner] = useState(false);
  const [likedPosts, setLikedPosts] = useState([]);
  const [showOwnerModal, setShowOwnerModal] = useState(false);
  const [ownerError, setOwnerError] = useState("");
  const [editMode, setEditMode] = useState(false);
  const [saving, setSaving] = useState(false);
  const [toast, setToast] = useState("");
  const [showColorPicker, setShowColorPicker] = useState(false);
  const [albumNavOpen, setAlbumNavOpen] = useState(false);
  const [jumpAlbumId, setJumpAlbumId] = useState(null);
  const [ageVerified, setAgeVerified] = useState(false);
  const [showNSFW, setShowNSFW] = useState(false);
  const accent = content.theme?.accent || "#FF6F91";

  useEffect(() => {
    (async () => {
      try {
        const { data, error } = await supabase.storage.from('images').download('site_content.json');
        if (data) {
          const text = await data.text();
          const parsed = JSON.parse(text);
          setContent({
            ...DEFAULT_CONTENT, ...parsed,
            contact: { ...DEFAULT_CONTENT.contact, ...(parsed.contact || {}) },
            nav: { ...DEFAULT_CONTENT.nav, ...(parsed.nav || {}) },
            theme: { ...DEFAULT_CONTENT.theme, ...(parsed.theme || {}) },
            tags: parsed.tags || [], hiddenTags: parsed.hiddenTags || [],
            posts: (parsed.posts || []).map((p) => ({ ...p, images: p.images?.length ? p.images : (p.image ? [p.image] : []) })),
          });
        }
        
        if (localStorage.getItem("personal:is-owner") === "true") setIsOwner(true);
        const savedLikes = localStorage.getItem("personal:liked-posts");
        if (savedLikes) setLikedPosts(JSON.parse(savedLikes));
        if (localStorage.getItem("personal:age-verified") === "true") setAgeVerified(true);
        if (localStorage.getItem("personal:show-nsfw") === "true") setShowNSFW(true);
      } catch (e) {
        console.error("load error", e);
      }
      setLoading(false);
    })();
  }, []);

  const confirmAge = () => { setAgeVerified(true); localStorage.setItem("personal:age-verified", "true"); };
  const toggleShowNSFW = () => { const next = !showNSFW; setShowNSFW(next); localStorage.setItem("personal:show-nsfw", String(next)); };
  const flashToast = (msg) => { setToast(msg); setTimeout(() => setToast(""), 1800); };

  const saveContent = useCallback(async (next) => {
    setContent(next);
    setSaving(true);
    try {
      const blob = new Blob([JSON.stringify(next)], { type: 'application/json' });
      await supabase.storage.from('images').upload('site_content.json', blob, { upsert: true, contentType: 'application/json', cacheControl: '0' });
    } catch (e) { console.error("save error", e); }
    setSaving(false);
  }, []);

  const handleLogin = (pw) => {
    if (pw === OWNER_PASSWORD) {
      setIsOwner(true); setEditMode(true); setShowOwnerModal(false); setOwnerError(""); flashToast("Edit mode enabled");
      localStorage.setItem("personal:is-owner", "true");
    } else { setOwnerError("Incorrect password"); }
  };
  const handleLogout = () => { setIsOwner(false); setEditMode(false); localStorage.setItem("personal:is-owner", "false"); };

  const logoFileRef = useRef(null);
  const uploadLogo = async (e) => {
    const file = e.target.files?.[0]; if (!file) return;
    setSaving(true);
    const url = await uploadToSupabase(file, 300);
    await saveContent({ ...content, nav: { ...content.nav, logo: url } });
    setSaving(false);
  };

  const toggleLike = async (postId) => {
    const liked = likedPosts.includes(postId);
    const newLiked = liked ? likedPosts.filter((id) => id !== postId) : [...likedPosts, postId];
    setLikedPosts(newLiked); localStorage.setItem("personal:liked-posts", JSON.stringify(newLiked));
    const newPosts = content.posts.map((p) => p.id === postId ? { ...p, likes: Math.max(0, p.likes + (liked ? -1 : 1)) } : p );
    await saveContent({ ...content, posts: newPosts });
  };

  const addComment = async (postId, name, text) => {
    const newPosts = content.posts.map((p) => p.id === postId ? { ...p, comments: [...p.comments, { id: uid(), name: name || "Guest", text }] } : p );
    await saveContent({ ...content, posts: newPosts });
  };

  if (loading) return <div className="min-h-screen flex items-center justify-center c-bg-FFFFFF"><Loader2 className="animate-spin c-text-262626" size={28} /></div>;
  if (!ageVerified) return <div className="min-h-screen c-bg-FFFFFF font-body c-text-1A1A1A"><FontStyle /><AgeGateModal onConfirm={confirmAge} accent={accent} /></div>;

  return (
    <div className="min-h-screen c-bg-FFFFFF font-body c-text-1A1A1A">
      <FontStyle />
      <nav className="sticky top-0 z-40 c-bg-FFFFFF-90 backdrop-blur border-b c-border-EBEBEB">
        <div className="max-w-5xl mx-auto px-4 py-3 flex items-center justify-between">
          <div className="flex items-center gap-1.5 font-display font-semibold c-text-1A1A1A">
            {isOwner && editMode ? (
              <button onClick={() => logoFileRef.current?.click()} className="relative flex items-center justify-center w-6 h-6 rounded-full overflow-hidden hover:opacity-80 transition-opacity">
                {content.nav?.logo ? <img src={content.nav.logo} className="w-full h-full object-cover" /> : <Sparkles size={16} style={{ color: accent }} />}
                <span className="absolute inset-0 bg-black/30 opacity-0 hover:opacity-100 flex items-center justify-center"><Pencil size={9} className="text-white" /></span>
              </button>
            ) : content.nav?.logo ? <img src={content.nav.logo} className="w-6 h-6 rounded-full object-cover" /> : <Sparkles size={16} style={{ color: accent }} />}
            <input ref={logoFileRef} type="file" accept="image/*" className="hidden" onChange={uploadLogo} />
            <span className="text-sm sm:text-base">{content.contact.name || "mini site"}</span>
          </div>
          <div className="flex items-center gap-1 sm:gap-2">
            {[{ id: "home", label: "Home", icon: HomeIcon }, { id: "feed", label: "Feed", icon: LayoutGrid }].map((t) => (
              <button key={t.id} onClick={() => setActiveTab(t.id)} style={activeTab === t.id ? { backgroundColor: accent } : undefined} className={`flex items-center gap-1 px-2.5 py-1.5 rounded-full text-xs sm:text-sm transition-colors ${activeTab === t.id ? "text-white" : "c-text-6B6B6B hoverc-bg-F2F2F2"}`}>
                <t.icon size={14} /><span className="hidden sm:inline">{t.label}</span>
              </button>
            ))}
            <div className="relative flex items-center">
              <button onClick={() => setActiveTab("album")} style={activeTab === "album" ? { backgroundColor: accent } : undefined} className={`flex items-center gap-1 pl-2.5 pr-1 py-1.5 rounded-full text-xs sm:text-sm transition-colors ${activeTab === "album" ? "text-white" : "c-text-6B6B6B hoverc-bg-F2F2F2"}`}>
                <Images size={14} /><span className="hidden sm:inline">Albums</span>
              </button>
              {content.albums.length > 0 && <button onClick={() => setAlbumNavOpen(!albumNavOpen)} className={`p-1 rounded-full ${activeTab === "album" ? "text-white/80 hover:text-white" : "c-text-A3A3A3 hoverc-bg-F2F2F2"}`}><ChevronDown size={12} /></button>}
              {albumNavOpen && (
                <div className="absolute z-40 top-9 left-0 bg-white rounded-xl shadow-lg border c-border-E0E0E0 p-1.5 w-44 max-h-64 overflow-y-auto">
                  {content.albums.map((a) => (<button key={a.id} onClick={() => { setActiveTab("album"); setJumpAlbumId(a.id); setAlbumNavOpen(false); }} className="w-full text-left text-xs px-2.5 py-1.5 rounded-lg hoverc-bg-F2F2F2 c-text-1A1A1A truncate">{a.name}</button>))}
                </div>
              )}
            </div>
            <button onClick={() => setActiveTab("contact")} style={activeTab === "contact" ? { backgroundColor: accent } : undefined} className={`flex items-center gap-1 px-2.5 py-1.5 rounded-full text-xs sm:text-sm transition-colors ${activeTab === "contact" ? "text-white" : "c-text-6B6B6B hoverc-bg-F2F2F2"}`}>
              <CreditCard size={14} /><span className="hidden sm:inline">Contact</span>
            </button>
            {isOwner && <button onClick={() => setEditMode(!editMode)} style={editMode ? { color: accent, borderColor: accent } : undefined} className={`ml-1 flex items-center gap-1 px-2.5 py-1.5 rounded-full text-xs sm:text-sm border ${editMode ? "bg-white" : "c-border-E0E0E0 c-text-6B6B6B hoverc-bg-F2F2F2"}`}><Pencil size={13} /><span className="hidden sm:inline">{editMode ? "Editing" : "Edit site"}</span></button>}
            <button onClick={isOwner ? handleLogout : () => setShowOwnerModal(true)} className="ml-1 p-2 rounded-full c-text-6B6B6B hoverc-bg-F2F2F2">{isOwner ? <Unlock size={15} /> : <Lock size={15} />}</button>
            {isOwner && <div className="relative"><button onClick={() => setShowColorPicker(!showColorPicker)} className="ml-1 p-2 rounded-full c-text-6B6B6B hoverc-bg-F2F2F2"><Palette size={15} /></button>{showColorPicker && <ThemeColorPicker accent={accent} onChange={(c) => saveContent({ ...content, theme: { ...content.theme, accent: c } })} onClose={() => setShowColorPicker(false)} />}</div>}
          </div>
        </div>
      </nav>

      {saving && <div className="fixed top-16 right-4 z-50 text-xs bg-white/90 shadow px-3 py-1.5 rounded-full c-text-6B6B6B flex items-center gap-1.5"><Loader2 size={12} className="animate-spin" /> Saving...</div>}
      {toast && <div className="fixed top-16 left-1/2 -translate-x-1/2 z-50 text-sm c-bg-1A1A1A text-white px-4 py-2 rounded-full shadow-lg">{toast}</div>}

      {activeTab === "home" && <HomePage content={content} saveContent={saveContent} editMode={editMode && isOwner} goFeed={() => setActiveTab("feed")} goAlbum={() => setActiveTab("album")} />}
      {activeTab === "feed" && <FeedPage content={content} saveContent={saveContent} editMode={editMode && isOwner} isOwner={isOwner} likedPosts={likedPosts} toggleLike={toggleLike} addComment={addComment} flashToast={flashToast} showNSFW={showNSFW} toggleShowNSFW={toggleShowNSFW} />}
      {activeTab === "album" && <AlbumPage content={content} saveContent={saveContent} editMode={editMode && isOwner} scrollToAlbumId={jumpAlbumId} onScrolled={() => setJumpAlbumId(null)} showNSFW={showNSFW} toggleShowNSFW={toggleShowNSFW} />}
      {activeTab === "contact" && <ContactPage content={content} saveContent={saveContent} editMode={editMode && isOwner} />}
      {showOwnerModal && <OwnerModal onClose={() => setShowOwnerModal(false)} onLogin={handleLogin} error={ownerError} accent={accent} />}
      
      <footer className="text-center text-xs c-text-A3A3A3 py-8 px-4">{content.footer}</footer>
    </div>
  );
}

function HomePage({ content, saveContent, editMode, goFeed, goAlbum }) {
  const { profile } = content; const accent = content.theme?.accent || "#FF6F91";
  const fileRef = useRef(null); const [draft, setDraft] = useState(profile);
  useEffect(() => setDraft(profile), [profile]);

  const upload = async (e) => {
    const file = e.target.files?.[0]; if (!file) return;
    const url = await uploadToSupabase(file, 1600);
    await saveContent({ ...content, profile: { ...content.profile, coverImage: url } });
  };

  return (
    <div className="relative">
      <div className="relative min-h-[80vh] flex items-center justify-center overflow-hidden" style={{ backgroundImage: profile.coverImage ? `url(${profile.coverImage})` : undefined, backgroundColor: profile.coverImage ? undefined : "#F2F2F2", backgroundSize: "cover", backgroundPosition: "center" }}>
        <div className="absolute inset-0 cover-gradient" />
        <div className="absolute -top-10 -left-10 w-40 h-40 blob" style={{ backgroundColor: accent, opacity: 0.3 }} />
        <div className="absolute bottom-10 -right-6 w-28 h-28 c-bg-E5E5E5-40 blob" />
        <div className="relative z-10 text-center px-6 max-w-lg">
          {editMode ? <input value={draft.eyebrow} onChange={(e) => setDraft({ ...draft, eyebrow: e.target.value })} onBlur={() => saveContent({...content, profile: draft})} className="text-center bg-white/80 rounded-full px-3 py-1 text-sm mb-3 font-body w-full" /> : <p className="text-sm font-body text-white drop-shadow mb-3">{profile.eyebrow}</p>}
          {editMode ? <textarea value={draft.title} onChange={(e) => setDraft({ ...draft, title: e.target.value })} onBlur={() => saveContent({...content, profile: draft})} rows={2} className="text-center bg-white/85 rounded-2xl px-3 py-2 font-display font-semibold text-2xl sm:text-4xl w-full mb-3" /> : <h1 className="font-display font-semibold text-3xl sm:text-5xl text-white drop-shadow-md mb-3">{profile.title}</h1>}
          {editMode ? <textarea value={draft.subtitle} onChange={(e) => setDraft({ ...draft, subtitle: e.target.value })} onBlur={() => saveContent({...content, profile: draft})} rows={2} className="text-center bg-white/80 rounded-2xl px-3 py-2 font-body text-sm w-full mb-5" /> : <p className="font-body text-white/90 drop-shadow mb-6">{profile.subtitle}</p>}
          <button onClick={goFeed} style={{ color: accent }} className="gentle-bounce inline-flex items-center gap-2 bg-white font-display font-medium px-6 py-3 rounded-full shadow-lg hoverc-bg-F2F2F2">{profile.buttonText}</button>
        </div>
        {editMode && <div className="absolute top-4 right-4 z-20"><button onClick={() => fileRef.current?.click()} className="flex items-center gap-1.5 bg-white/90 px-3 py-2 rounded-full text-xs shadow"><ImageIcon size={14} /> Change cover photo</button><input ref={fileRef} type="file" accept="image/*" onChange={upload} className="hidden" /></div>}
      </div>
      <div className="dashed-divider max-w-5xl mx-auto" />
      {content.posts.length > 0 && (
        <div className="max-w-5xl mx-auto px-4 py-10">
          <div className="flex items-center justify-between mb-4"><h2 className="font-display font-semibold text-lg sm:text-xl">Scrollable Feed</h2><button onClick={goFeed} className="text-xs sm:text-sm font-body c-text-6B6B6B flex items-center gap-1">View all posts <ChevronRight size={14} /></button></div>
          <div className="flex gap-3 overflow-x-auto pb-2 scrollbar-thin snap-x">
            {content.posts.slice(0, 10).map((post) => (
              <button key={post.id} onClick={goFeed} className="text-left flex-shrink-0 w-40 sm:w-48 snap-start group">
                <div className="relative w-40 h-40 sm:w-48 sm:h-48 rounded-2xl overflow-hidden c-bg-F2F2F2"><img src={post.images[0]} className={`w-full h-full object-cover transition-transform ${post.nsfw ? "blur-xl scale-110" : "group-hover:scale-105"}`} />{post.nsfw && <span className="absolute top-1.5 left-1.5 bg-black/60 text-white text-[10px] px-1.5 py-0.5 rounded-full">🔞 NSFW</span>}</div>
                <p className="text-xs c-text-4B4B4B mt-1.5 line-clamp-2">{post.caption || "No caption"}</p>
              </button>
            ))}
          </div>
        </div>
      )}
    </div>
  );
}

function FeedPage({ content, saveContent, editMode, isOwner, likedPosts, toggleLike, addComment, flashToast, showNSFW, toggleShowNSFW }) {
  const accent = content.theme?.accent || "#FF6F91";
  const [openPost, setOpenPost] = useState(null); const [showAdd, setShowAdd] = useState(false); const [tagFilter, setTagFilter] = useState(null);
  const [confirmDeleteId, setConfirmDeleteId] = useState(null); const [revealedIds, setRevealedIds] = useState([]);
  
  const allTags = [...new Set(content.posts.flatMap((p) => p.tags))];
  const tagMatched = tagFilter ? content.posts.filter((p) => p.tags.includes(tagFilter)) : content.posts;
  const visiblePosts = showNSFW ? tagMatched : tagMatched.filter((p) => !p.nsfw);

  return (
    <div className="max-w-5xl mx-auto px-4 py-8 grid grid-cols-1 sm:grid-cols-[1fr_260px] gap-8">
      <div>
        <div className="flex items-center justify-between mb-5">
          <h2 className="font-display font-semibold text-2xl">Feed</h2>
          {isOwner && <button onClick={() => setShowAdd(true)} style={{ backgroundColor: accent }} className="flex items-center gap-1.5 text-white px-3.5 py-2 rounded-full text-sm hover:opacity-90"><Plus size={15} /> Add Post</button>}
        </div>
        {tagFilter && <div className="mb-4"><Chip active accent={accent} onClick={() => setTagFilter(null)}>{tagLabel(tagFilter)} <X size={12} className="inline ml-1" /></Chip></div>}
        {visiblePosts.length === 0 ? <div className="text-center py-20 c-text-A3A3A3 text-sm">No posts yet</div> : (
          <div className="grid grid-cols-2 sm:grid-cols-3 gap-2.5">
            {visiblePosts.map((post) => {
              const needsReveal = post.nsfw && !revealedIds.includes(post.id);
              return (
                <div key={post.id} onClick={() => (needsReveal ? setRevealedIds([...revealedIds, post.id]) : setOpenPost(post))} className="relative aspect-square rounded-2xl overflow-hidden cursor-pointer group c-bg-F2F2F2">
                  <img src={post.images[0]} className={`w-full h-full object-cover transition-transform ${needsReveal ? "blur-xl scale-110" : "group-hover:scale-105"}`} />
                  {needsReveal ? <div className="absolute inset-0 flex flex-col items-center justify-center bg-black/30 text-white text-center px-2"><span className="text-lg">🔞</span><span className="text-[11px] font-body">NSFW — tap</span></div> : <>
                    {post.nsfw && <span className="absolute top-1.5 left-1.5 bg-black/60 text-white text-[10px] px-1.5 py-0.5 rounded-full">🔞 NSFW</span>}
                    {editMode && <button onClick={(e) => { e.stopPropagation(); setConfirmDeleteId(post.id); }} className="absolute top-1.5 right-1.5 bg-white/90 p-1.5 rounded-full text-rose-600 opacity-0 group-hover:opacity-100"><Trash2 size={13} /></button>}
                  </>}
                </div>
              );
            })}
          </div>
        )}
      </div>
      <div className="space-y-6 sm:sticky sm:top-20 sm:self-start">
        <div className="bg-white/70 rounded-2xl p-4 border c-border-EBEBEB">
          <button onClick={toggleShowNSFW} className="w-full flex items-center justify-between gap-2"><span className="text-sm font-body c-text-1A1A1A">🔞 Show NSFW content</span><span className={`relative w-9 h-5 rounded-full transition-colors flex-shrink-0 ${showNSFW ? "" : "c-bg-F2F2F2"}`} style={showNSFW ? { backgroundColor: accent } : undefined}><span className={`absolute top-0.5 w-4 h-4 rounded-full bg-white shadow transition-all ${showNSFW ? "left-4" : "left-0.5"}`} /></span></button>
        </div>
        <div className="bg-white/70 rounded-2xl p-4 border c-border-EBEBEB"><h3 className="font-display font-semibold text-sm mb-3 flex items-center gap-1.5"><Tag size={14} /> All Tags</h3>
          <div className="flex flex-wrap gap-1.5">{allTags.map((t) => (<Chip key={t} active={tagFilter === t} accent={accent} onClick={() => setTagFilter(tagFilter === t ? null : t)}>{tagLabel(t)}</Chip>))}</div>
        </div>
      </div>
      {openPost && <PostModal post={content.posts.find(p=>p.id===openPost.id)||openPost} onClose={() => setOpenPost(null)} liked={likedPosts.includes(openPost.id)} onLike={() => toggleLike(openPost.id)} onComment={(name, text) => addComment(openPost.id, name, text)} isOwner={isOwner} onDelete={() => setConfirmDeleteId(openPost.id)} accent={accent} />}
      {showAdd && <AddPostModal content={content} saveContent={saveContent} onClose={() => setShowAdd(false)} onDone={() => flashToast("Post added ✿")} />}
      {confirmDeleteId && <ConfirmModal message="Delete this post?" accent={accent} onCancel={() => setConfirmDeleteId(null)} onConfirm={() => { saveContent({ ...content, posts: content.posts.filter((p) => p.id !== confirmDeleteId) }); setOpenPost(null); setConfirmDeleteId(null); }} />}
    </div>
  );
}

function PostModal({ post, onClose, liked, onLike, onComment, isOwner, onDelete, accent }) {
  const [name, setName] = useState(""); const [text, setText] = useState(""); const [imgIndex, setImgIndex] = useState(0);
  const images = post.images && post.images.length ? post.images : [post.image];
  return (
    <div className="fixed inset-0 z-50 backdrop-blur-sm flex items-center justify-center p-4" style={{ backgroundColor: "rgba(0,0,0,0.5)" }}>
      <div className="c-bg-FFFFFF rounded-3xl w-full max-w-3xl max-h-[90vh] overflow-hidden grid grid-cols-1 md:grid-cols-2 relative shadow-2xl">
        <button onClick={onClose} className="absolute top-3 right-3 z-10 bg-white/90 p-1.5 rounded-full"><X size={16} /></button>
        <div className="bg-black flex items-center justify-center relative"><img src={images[imgIndex]} className="max-h-[45vh] md:max-h-[90vh] w-full object-cover" /></div>
        <div className="p-5 flex flex-col overflow-y-auto scrollbar-thin">
          <p className="font-body text-sm mb-2">{post.caption}</p>
          <div className="flex items-center gap-3 mb-4"><button onClick={onLike} className={`flex items-center gap-1.5 px-3 py-1.5 rounded-full text-sm border transition-colors ${liked ? "bg-rose-50 border-rose-300 text-rose-600" : "c-border-E0E0E0 c-text-6B6B6B"}`}><Heart size={14} fill={liked ? "currentColor" : "none"} /> {post.likes}</button>{isOwner && <button onClick={onDelete} className="ml-auto text-rose-500 flex items-center gap-1 text-xs"><Trash2 size={13} /></button>}</div>
          <div className="border-t border-b c-border-EBEBEB py-3 space-y-2 mb-3">
            <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Your name" className="w-full rounded-xl border c-border-E0E0E0 px-3 py-1.5 text-sm bg-white" />
            <div className="flex gap-2"><input value={text} onChange={(e) => setText(e.target.value)} placeholder="Write a comment..." className="flex-1 rounded-xl border c-border-E0E0E0 px-3 py-1.5 text-sm bg-white" onKeyDown={(e) => { if (e.key === "Enter" && text.trim()) { onComment(name.trim(), text.trim()); setText(""); } }} /><button onClick={() => { if(text.trim()) {onComment(name.trim(), text.trim()); setText("");} }} style={{ backgroundColor: accent }} className="text-white px-3 rounded-xl text-sm">Send</button></div>
          </div>
          <div className="flex-1 space-y-2.5">
            {[...post.comments].reverse().map((c) => <div key={c.id} className="text-sm"><span className="font-medium c-text-1A1A1A">{c.name}</span> <span className="c-text-4B4B4B">{c.text}</span></div>)}
          </div>
        </div>
      </div>
    </div>
  );
}

function AddPostModal({ content, saveContent, onClose, onDone }) {
  const accent = content.theme?.accent || "#FF6F91";
  const [images, setImages] = useState([]); const [caption, setCaption] = useState(""); const [tags, setTags] = useState([]);
  const [tagInput, setTagInput] = useState(""); const [tagEmoji, setTagEmoji] = useState(TAG_EMOJIS[0]);
  const [busy, setBusy] = useState(false); const [nsfw, setNsfw] = useState(false);

  const handleFiles = async (e) => {
    const files = Array.from(e.target.files || []); if (!files.length) return;
    setBusy(true);
    try {
      const uploadedUrls = await Promise.all(files.slice(0, 6 - images.length).map(f => uploadToSupabase(f, 1400)));
      setImages((prev) => [...prev, ...uploadedUrls]);
    } catch (err) { alert("Upload failed"); }
    setBusy(false); e.target.value = "";
  };

  const submit = async () => {
    if (images.length === 0) return;
    const newPost = { id: uid(), images, caption, tags, likes: 0, comments: [], nsfw };
    await saveContent({ ...content, posts: [newPost, ...content.posts] });
    onDone(); onClose();
  };

  return (
    <div className="fixed inset-0 z-50 backdrop-blur-sm flex items-center justify-center p-4" style={{ backgroundColor: "rgba(0,0,0,0.5)" }}>
      <div className="c-bg-FFFFFF rounded-3xl p-6 w-full max-w-md shadow-2xl relative max-h-[90vh] overflow-y-auto">
        <button onClick={onClose} className="absolute top-4 right-4 c-text-6B6B6B"><X size={18} /></button>
        <h3 className="font-display font-semibold text-lg mb-4">Add New Post</h3>
        <FieldLabel>Photos</FieldLabel>
        <div className="grid grid-cols-3 gap-2 mb-3">
          {images.map((img, idx) => (<div key={idx} className="relative aspect-square rounded-xl overflow-hidden c-bg-F2F2F2"><img src={img} className="w-full h-full object-cover" /><button onClick={() => setImages(images.filter((_, i) => i !== idx))} className="absolute top-1 right-1 bg-white/90 rounded-full p-0.5 text-rose-600"><X size={12} /></button></div>))}
          {images.length < 6 && (
            <label className="aspect-square rounded-xl border-2 border-dashed c-border-E0E0E0 flex items-center justify-center c-text-A3A3A3 text-xs cursor-pointer hoverc-bg-F2F2F2">
              {busy ? <Loader2 className="animate-spin" size={18} /> : <span className="flex flex-col items-center gap-1"><Plus size={16} /> Add Photo</span>}
              <input type="file" accept="image/*" multiple className="hidden" onChange={handleFiles} />
            </label>
          )}
        </div>
        <FieldLabel>Caption</FieldLabel>
        <textarea value={caption} onChange={(e) => setCaption(e.target.value)} rows={2} className="w-full rounded-xl border c-border-E0E0E0 px-3 py-2 mb-3 bg-white text-sm" />
        
        <button type="button" onClick={() => setNsfw(!nsfw)} className={`w-full flex items-center justify-between gap-2 rounded-xl border px-3 py-2.5 mb-4 transition-colors ${nsfw ? "border-transparent" : "c-border-E0E0E0"}`} style={nsfw ? { backgroundColor: "#FFF1F2", borderColor: "#FDA4AF" } : undefined}>
          <span className="flex items-center gap-1.5 text-sm font-body c-text-1A1A1A">🔞 Mark as NSFW</span>
          <span className={`relative w-9 h-5 rounded-full transition-colors flex-shrink-0 ${nsfw ? "" : "c-bg-F2F2F2"}`} style={nsfw ? { backgroundColor: accent } : undefined}><span className={`absolute top-0.5 w-4 h-4 rounded-full bg-white shadow transition-all ${nsfw ? "left-4" : "left-0.5"}`} /></span>
        </button>
        <button onClick={submit} disabled={images.length === 0} style={{ backgroundColor: accent }} className="w-full py-2.5 rounded-xl text-white font-medium disabled:opacity-40 hover:opacity-90">Post</button>
      </div>
    </div>
  );
}

function AlbumPage({ content, saveContent, editMode, showNSFW }) {
  const accent = content.theme?.accent || "#FF6F91";
  const [showAddAlbum, setShowAddAlbum] = useState(false);
  const visibleAlbums = showNSFW ? content.albums : content.albums.filter((a) => !a.nsfw);

  const addImages = async (albumId, files) => {
    const uploadedUrls = await Promise.all(Array.from(files).map((f) => uploadToSupabase(f, 1400)));
    const newImgs = uploadedUrls.map((src) => ({ id: uid(), src }));
    const albums = content.albums.map((a) => a.id === albumId ? { ...a, images: [...a.images, ...newImgs] } : a );
    await saveContent({ ...content, albums });
  };

  return (
    <div className="max-w-5xl mx-auto px-4 py-8">
      <div className="flex items-center justify-between mb-6 gap-3">
        <h2 className="font-display font-semibold text-2xl">Photo Albums</h2>
        {editMode && <button onClick={() => setShowAddAlbum(true)} style={{ backgroundColor: accent }} className="flex items-center gap-1.5 text-white px-3.5 py-2 rounded-full text-sm hover:opacity-90"><Plus size={15} /> New Album</button>}
      </div>
      <div className="space-y-9">
        {visibleAlbums.map((album) => (
          <div key={album.id}>
            <div className="flex items-center justify-between mb-3">
              <h3 className="font-display font-medium text-lg flex items-center gap-1.5">{album.name} {album.nsfw && <span className="text-[10px] border c-border-D0D0D0 px-1.5 py-0.5 rounded-full">🔞 NSFW</span>}</h3>
              {editMode && (
                <div className="flex items-center gap-2">
                  <label className="flex items-center gap-1 text-xs bg-white border c-border-E0E0E0 px-2.5 py-1.5 rounded-full cursor-pointer hoverc-bg-F2F2F2"><Plus size={12} /> Add Photos<input type="file" accept="image/*" multiple className="hidden" onChange={(e) => e.target.files?.length && addImages(album.id, e.target.files)} /></label>
                  <button onClick={() => saveContent({ ...content, albums: content.albums.filter((a) => a.id !== album.id) })} className="text-rose-500"><Trash2 size={14} /></button>
                </div>
              )}
            </div>
            <div className="flex gap-3 overflow-x-auto pb-2 scrollbar-thin snap-x">
              {album.images.map((img) => (<div key={img.id} className="relative flex-shrink-0 snap-start group"><img src={img.src} className="h-40 w-40 object-cover rounded-2xl" />{editMode && <button onClick={() => saveContent({ ...content, albums: content.albums.map((a) => a.id === album.id ? { ...a, images: a.images.filter((im) => im.id !== img.id) } : a) })} className="absolute top-1.5 right-1.5 bg-white/90 p-1.5 rounded-full text-rose-600 opacity-0 group-hover:opacity-100"><Trash2 size={12} /></button>}</div>))}
            </div>
          </div>
        ))}
      </div>
      {showAddAlbum && <AddAlbumModal accent={accent} onClose={() => setShowAddAlbum(false)} onSubmit={(name, nsfw) => { saveContent({ ...content, albums: [...content.albums, { id: uid(), name, images: [], nsfw }] }); setShowAddAlbum(false); }} />}
    </div>
  );
}

function ContactPage({ content, saveContent, editMode }) {
  const accent = content.theme?.accent || "#FF6F91";
  const [draft, setDraft] = useState(content.contact); const fileRef = useRef(null);
  const commit = async (updated) => { await saveContent({ ...content, contact: updated ?? draft }); };
  const uploadAvatar = async (e) => {
    const file = e.target.files?.[0]; if (!file) return;
    const url = await uploadToSupabase(file, 600);
    const updated = { ...content.contact, avatar: url }; setDraft(updated); await commit(updated);
  };

  return (
    <div className="max-w-lg mx-auto px-4 py-14">
      <div className="bg-white rounded-[2rem] shadow-lg border c-border-EBEBEB p-8 relative overflow-hidden text-center sm:text-left">
        <div className="relative inline-block mx-auto sm:mx-0 mb-4">
          <div className="w-24 h-24 overflow-hidden rounded-2xl c-bg-F2F2F2">{draft.avatar ? <img src={draft.avatar} className="w-full h-full object-cover" /> : <div className="w-full h-full flex items-center justify-center c-text-B0B0B0 text-3xl font-display">?</div>}</div>
          {editMode && <><button onClick={() => fileRef.current?.click()} style={{ backgroundColor: accent }} className="absolute -bottom-1 -right-1 text-white p-1.5 rounded-full shadow hover:opacity-90"><Pencil size={11} /></button><input ref={fileRef} type="file" accept="image/*" className="hidden" onChange={uploadAvatar} /></>}
        </div>
        {editMode ? <input value={draft.name} onChange={(e) => setDraft({ ...draft, name: e.target.value })} onBlur={() => commit()} className="font-display font-semibold text-xl w-full mb-1 c-bg-FFFFFF rounded-lg" /> : <h2 className="font-display font-semibold text-xl mb-1">{draft.name}</h2>}
        {editMode ? <textarea value={draft.bio} onChange={(e) => setDraft({ ...draft, bio: e.target.value })} onBlur={() => commit()} rows={2} className="text-sm w-full mb-5 c-bg-FFFFFF rounded-lg px-2 py-1" /> : <p className="text-sm mb-5 c-text-4B4B4B">{draft.bio}</p>}
      </div>
    </div>
  );
}
