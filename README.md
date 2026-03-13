import React, { useState, useEffect } from 'react';
import { initializeApp } from 'firebase/app';
import { 
  getFirestore, 
  doc, 
  setDoc, 
  onSnapshot, 
  collection, 
  query, 
  addDoc 
} from 'firebase/firestore';
import { 
  getAuth, 
  signInAnonymously, 
  signInWithCustomToken, 
  onAuthStateChanged 
} from 'firebase/auth';
import { 
  LayoutGrid, 
  Fingerprint, 
  PlayCircle, 
  Target, 
  TrendingUp, 
  Users, 
  Search,
  RefreshCw,
  ShieldCheck,
  Zap,
  MessageSquare,
  ExternalLink,
  Radio
} from 'lucide-react';

// --- Firebase Configuration (Outside Component) ---
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'squad-hq-ultimate';

const DISCORD_LINK = "https://discord.gg/ecTy73BDY";

const App = () => {
  const [user, setUser] = useState(null);
  const [activeTab, setActiveTab] = useState('overview');
  const [intel, setIntel] = useState([
    { weapon: "SVA 545", build: "Muzzle: Spiritfire, Barrel: STV Precision, Optic: Corio Eagleseye", tier: "META" },
    { weapon: "RAM-7", build: "Muzzle: Casus Brake, Barrel: Cronen Ritual, Stock: HVS Pad", tier: "META" }
  ]);
  const [ytSearch, setYtSearch] = useState('Warzone 3 Meta Loadouts');
  const [ytUrl, setYtUrl] = useState('https://www.youtube.com/embed/videoseries?list=PLPZ7UisxNf_8R6U1Y_mE2uLh7n0P0vYk4');
  const [time, setTime] = useState(new Date().toLocaleTimeString());
  const [isLoadingIntel, setIsLoadingIntel] = useState(false);

  // --- Auth Initialization (RULE 3) ---
  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (err) {
        console.error("Auth Error:", err);
      }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
    });
    
    const timer = setInterval(() => setTime(new Date().toLocaleTimeString()), 1000);
    return () => {
      unsubscribe();
      clearInterval(timer);
    };
  }, []);

  // --- AI Intel Fetcher (Exponential Backoff & Error Safety) ---
  const fetchIntel = async (retryCount = 0) => {
    setIsLoadingIntel(true);
    const apiKey = ""; // Provided by env
    const prompt = "Return a JSON array of the current top 4 Warzone meta weapons. Format: [{weapon, build, tier}]. Tier should be S-TIER or A-TIER.";
    
    try {
      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts: [{ text: prompt }] }],
          generationConfig: { responseMimeType: "application/json" }
        })
      });

      if (!response.ok && retryCount < 5) {
        const delay = Math.pow(2, retryCount) * 1000;
        setTimeout(() => fetchIntel(retryCount + 1), delay);
        return;
      }

      const data = await response.json();
      const rawText = data.candidates?.[0]?.content?.parts?.[0]?.text;
      if (rawText) {
        const parsed = JSON.parse(rawText);
        setIntel(Array.isArray(parsed) ? parsed : intel);
      }
    } catch (e) {
      console.error("AI Intel Error", e);
    } finally {
      setIsLoadingIntel(false);
    }
  };

  useEffect(() => {
    fetchIntel();
  }, []);

  const NavItem = ({ id, icon: Icon, label }) => (
    <button 
      onClick={() => setActiveTab(id)}
      className={`p-4 flex flex-col items-center gap-1 transition-all border-l-2 ${activeTab === id ? 'text-emerald-500 border-emerald-500 bg-emerald-500/10' : 'text-slate-500 border-transparent hover:text-emerald-400 hover:bg-white/5'}`}
    >
      <Icon size={20} />
      <span className="text-[9px] font-bold uppercase tracking-tighter" style={{ fontFamily: 'monospace' }}>{label}</span>
    </button>
  );

  return (
    <div className="flex h-screen w-full bg-[#050506] text-slate-200 overflow-hidden font-sans select-none">
      
      {/* SIDEBAR */}
      <nav className="w-20 border-r border-white/5 bg-black flex flex-col py-8 z-50">
        <div className="mb-10 flex justify-center">
          <div className="w-8 h-8 border-2 border-emerald-500 flex items-center justify-center rotate-45 shadow-[0_0_15px_rgba(16,185,129,0.4)]">
            <div className="w-3 h-3 bg-emerald-500 rotate-[-45deg]"></div>
          </div>
        </div>
        <div className="flex flex-col gap-2">
          <NavItem id="overview" icon={LayoutGrid} label="HUD" />
          <NavItem id="intel" icon={Fingerprint} label="INTEL" />
          <NavItem id="media" icon={PlayCircle} label="MEDIA" />
          <NavItem id="training" icon={Target} label="SIM" />
        </div>
        
        <div className="mt-auto flex flex-col items-center pb-4">
           <a 
            href={DISCORD_LINK} 
            target="_blank" 
            rel="noopener noreferrer"
            className="p-4 text-slate-500 hover:text-[#5865F2] transition-all flex flex-col items-center gap-1 group"
           >
             <MessageSquare size={20} className="group-hover:drop-shadow-[0_0_8px_#5865F2]" />
             <span className="text-[8px] font-bold uppercase tracking-tighter">COMMS</span>
           </a>
        </div>
      </nav>

      {/* MAIN BODY */}
      <div className="flex-1 flex flex-col relative overflow-hidden">
        
        {/* TOP HUD */}
        <header className="h-16 border-b border-white/5 flex items-center justify-between px-8 bg-black/40 backdrop-blur-xl z-40">
          <div className="flex items-center gap-8">
            <div className="flex flex-col">
              <span className="text-[10px] text-emerald-500 font-bold uppercase tracking-[0.2em]" style={{ fontFamily: 'monospace' }}>Secure_Uplink</span>
              <span className="text-xs font-black tracking-tight uppercase">HQ // {activeTab.toUpperCase()}</span>
            </div>
            <div className="h-8 w-px bg-white/5"></div>
            <div className="flex items-center gap-2">
               <div className={`w-2 h-2 rounded-full ${user ? 'bg-emerald-500 animate-pulse' : 'bg-red-500'}`}></div>
               <span className="text-[10px] text-white uppercase opacity-70" style={{ fontFamily: 'monospace' }}>
                 {user ? `Unit_${user.uid.substring(0,6)}` : 'Syncing...'}
               </span>
            </div>
          </div>
          
          <div className="flex items-center gap-6">
             <div className="hidden md:flex flex-col items-end">
                <span className="text-[9px] text-slate-500 uppercase">Zulu_Time</span>
                <span className="text-sm font-bold text-white" style={{ fontFamily: 'monospace' }}>{time}</span>
             </div>
          </div>
        </header>

        {/* MARQUEE TICKER */}
        <div className="h-8 bg-emerald-500/5 border-b border-emerald-500/10 flex items-center overflow-hidden">
          <div className="flex whitespace-nowrap animate-marquee uppercase text-[10px] text-emerald-500 font-bold tracking-widest" style={{ fontFamily: 'monospace' }}>
             <span className="mx-12">UAV: $6,000 ▲</span>
             <span className="mx-12">Loadout: $10,000 (SOLO) ▼</span>
             <span className="mx-12">Precision: $4,500 ■</span>
             <span className="mx-12">Armor Box: $2,000 ▲</span>
             <span className="mx-12">Gas_Circle: 01:45 Remaining</span>
             <span className="mx-12">Buy Station Active ▲</span>
             <span className="mx-12">UAV: $6,000 ▲</span>
             <span className="mx-12">Loadout: $10,000 (SOLO) ▼</span>
             <span className="mx-12">Precision: $4,500 ■</span>
             <span className="mx-12">Armor Box: $2,000 ▲</span>
          </div>
        </div>

        <main className="flex-1 overflow-y-auto relative p-6">
          
          {activeTab === 'overview' && (
            <div className="grid grid-cols-12 gap-6 max-w-7xl mx-auto animate-in fade-in duration-500">
              {/* Mission Briefing */}
              <div className="col-span-12 lg:col-span-8 bg-[#0a0a0c] border border-white/5 rounded-2xl p-10 flex flex-col justify-center relative overflow-hidden group">
                <div className="absolute top-0 right-0 p-8 text-emerald-500/5 group-hover:text-emerald-500/10 transition-colors"><Zap size={200} /></div>
                <div className="relative z-10">
                  <div className="inline-flex items-center gap-2 mb-4 bg-emerald-500/10 px-3 py-1 rounded-full border border-emerald-500/20">
                    <Radio size={12} className="text-emerald-500" />
                    <span className="text-emerald-500 text-[10px] font-bold uppercase tracking-widest" style={{ fontFamily: 'monospace' }}>Status: Nominal</span>
                  </div>
                  <h2 className="text-6xl font-black text-white mb-6 tracking-tighter uppercase italic leading-none">Command_Terminal</h2>
                  <p className="text-slate-400 max-w-xl text-sm leading-relaxed mb-10">Integrated tactical environment for Warzone operatives. All intelligence feeds, media streams, and squad server links are active and encrypted.</p>
                  <div className="flex flex-wrap gap-4">
                    <button onClick={() => setActiveTab('intel')} className="px-8 py-4 bg-emerald-600 text-white rounded-lg font-bold text-xs uppercase tracking-widest hover:bg-emerald-500 transition-all flex items-center gap-3 shadow-[0_10px_30px_rgba(16,185,129,0.2)]">
                      <Fingerprint size={16} /> Open Intel Uplink
                    </button>
                    <a 
                      href={DISCORD_LINK} 
                      target="_blank" 
                      rel="noopener noreferrer"
                      className="px-8 py-4 bg-[#5865F2] text-white rounded-lg font-bold text-xs uppercase tracking-widest hover:bg-[#4752C4] transition-all flex items-center gap-3 shadow-[0_10px_30px_rgba(88,101,242,0.2)]"
                    >
                      <MessageSquare size={16} /> Join Discord
                    </a>
                  </div>
                </div>
              </div>

              {/* Status Widget */}
              <div className="col-span-12 lg:col-span-4 space-y-6">
                <div className="bg-[#0a0a0c] border border-white/5 rounded-2xl p-8">
                  <h3 className="text-[11px] font-bold text-slate-500 uppercase mb-6 flex justify-between" style={{ fontFamily: 'monospace' }}><span>Squad_Network</span><span className="text-emerald-500">CONNECTED</span></h3>
                  <div className="space-y-6">
                    <div className="flex items-center justify-between text-[12px]" style={{ fontFamily: 'monospace' }}>
                        <span className="text-slate-400">Node Latency</span>
                        <span className="text-emerald-500">14ms</span>
                    </div>
                    <div className="h-1.5 bg-white/5 rounded-full overflow-hidden">
                      <div className="h-full bg-emerald-500 w-[94%] animate-pulse"></div>
                    </div>
                    <div className="p-4 bg-white/5 rounded-xl border border-white/5">
                      <p className="text-[10px] text-slate-400 leading-relaxed uppercase" style={{ fontFamily: 'monospace' }}>
                        Primary comms channel active via Discord. Coordinates locked.
                      </p>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          )}

          {activeTab === 'intel' && (
            <div className="max-w-6xl mx-auto py-4 animate-in slide-in-from-bottom-4 duration-500">
               <div className="flex justify-between items-center mb-10 border-b border-white/5 pb-8">
                <div>
                  <h2 className="text-4xl font-black uppercase text-white tracking-tighter italic">Tactical_Intelligence</h2>
                  <p className="text-xs text-slate-500 uppercase mt-2" style={{ fontFamily: 'monospace' }}>Live metadata from global engagement zones</p>
                </div>
                <button 
                  onClick={() => fetchIntel()} 
                  disabled={isLoadingIntel}
                  className="p-4 bg-emerald-600 text-white rounded-xl hover:bg-emerald-500 transition-all disabled:opacity-50"
                >
                  <RefreshCw size={20} className={isLoadingIntel ? 'animate-spin' : ''} />
                </button>
              </div>
              <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                {intel.map((item, idx) => (
                  <div key={idx} className="bg-[#0a0a0c] border border-white/5 p-8 rounded-3xl relative overflow-hidden group hover:border-emerald-500/30 transition-all">
                    <div className="absolute top-0 right-0 p-4 opacity-5 group-hover:opacity-10 transition-all"><Fingerprint size={100} /></div>
                    <span className="text-[10px] text-emerald-500 font-bold uppercase tracking-widest" style={{ fontFamily: 'monospace' }}>{item.tier || 'CLASSIFIED'}</span>
                    <h3 className="text-4xl font-black text-white mt-2 mb-6 uppercase tracking-tight italic">{item.weapon}</h3>
                    <div className="bg-white/5 p-5 rounded-2xl border border-white/5">
                      <p className="text-[9px] text-slate-500 uppercase mb-3" style={{ fontFamily: 'monospace' }}>Config_Loadout</p>
                      <p className="text-sm text-emerald-400 font-bold leading-relaxed">{item.build}</p>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          )}

          {activeTab === 'media' && (
            <div className="max-w-[1600px] mx-auto h-full flex flex-col xl:flex-row gap-8 animate-in zoom-in-95 duration-500">
              <div className="flex-1 flex flex-col">
                <div className="flex items-center gap-3 mb-6">
                  <div className="w-10 h-10 bg-emerald-500/10 rounded-xl flex items-center justify-center text-emerald-500 border border-emerald-500/20"><PlayCircle size={20} /></div>
                  <h3 className="text-xs font-bold uppercase tracking-[0.3em]">Audio_Uplink</h3>
                </div>
                <div className="flex-1 bg-black rounded-3xl overflow-hidden border border-white/5 shadow-2xl min-h-[500px]">
                   <iframe style={{borderRadius:'12px'}} src="https://open.spotify.com/embed/playlist/37i9dQZF1DWWY64wDtewQt?utm_source=generator&theme=0" width="100%" height="100%" frameBorder="0" allowFullScreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>
                </div>
              </div>
              
              <div className="flex-1 flex flex-col">
                <div className="flex items-center gap-3 mb-6">
                  <div className="w-10 h-10 bg-red-600/10 rounded-xl flex items-center justify-center text-red-600 border border-red-600/20"><Search size={20} /></div>
                  <h3 className="text-xs font-bold uppercase tracking-[0.3em]">Visual_Feed</h3>
                </div>
                <div className="flex gap-2 mb-6">
                  <input 
                    type="text" 
                    value={ytSearch}
                    onChange={(e) => setYtSearch(e.target.value)}
                    onKeyDown={(e) => e.key === 'Enter' && setYtUrl(`https://www.youtube.com/embed?listType=search&list=${encodeURIComponent(ytSearch)}`)}
                    className="flex-1 bg-white/5 border border-white/10 rounded-xl px-6 py-3 text-xs focus:border-emerald-500 focus:outline-none transition-all"
                    placeholder="Search tactical feeds..."
                  />
                  <button 
                    onClick={() => setYtUrl(`https://www.youtube.com/embed?listType=search&list=${encodeURIComponent(ytSearch)}`)}
                    className="bg-white text-black px-8 py-3 rounded-xl text-[10px] font-black uppercase tracking-widest hover:bg-emerald-500 hover:text-white transition-all"
                  >
                    SCAN
                  </button>
                </div>
                <div className="flex-1 bg-black rounded-3xl overflow-hidden border border-white/5 shadow-2xl min-h-[400px]">
                  <iframe width="100%" height="100%" src={ytUrl} allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowFullScreen></iframe>
                </div>
              </div>
            </div>
          )}

          {activeTab === 'training' && (
             <div className="h-full w-full flex items-center justify-center animate-pulse">
                <div className="text-center p-16 bg-[#0a0a0c] border border-white/5 rounded-[3rem] max-w-lg">
                   <Target size={80} className="mx-auto text-emerald-500 mb-8" />
                   <h2 className="text-3xl font-black text-white uppercase mb-4 tracking-tighter italic">Calibration_Pending</h2>
                   <p className="text-xs text-slate-500 mb-10 leading-relaxed uppercase" style={{ fontFamily: 'monospace' }}>Neural training environment is currently offline for node maintenance. Sync with squad via Discord for live drills.</p>
                   <a href={DISCORD_LINK} target="_blank" rel="noopener noreferrer" className="inline-flex w-full py-5 bg-emerald-600 text-white font-bold rounded-2xl uppercase tracking-widest shadow-[0_0_30px_rgba(16,185,129,0.2)] hover:scale-[1.02] transition-transform">
                     Manual Uplink Required
                   </a>
                </div>
             </div>
          )}

        </main>
      </div>

      <style dangerouslySetInnerHTML={{ __html: `
        @keyframes marquee {
          0% { transform: translateX(0); }
          100% { transform: translateX(-50%); }
        }
        .animate-marquee {
          display: flex;
          width: 200%;
          animation: marquee 30s linear infinite;
        }
        ::-webkit-scrollbar { width: 5px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.05); border-radius: 10px; }
        ::-webkit-scrollbar-thumb:hover { background: rgba(16,185,129,0.2); }
      `}} />
    </div>
  );
};

export default App;


