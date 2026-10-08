# wakzil-studio
<!doctype html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Wakzil Studio</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  <style>
    html,body,#root{min-height:100%;margin:0} body{background:#0f1115}*{box-sizing:border-box}
    ::-webkit-scrollbar{width:10px;height:10px}::-webkit-scrollbar-track{background:#111318}::-webkit-scrollbar-thumb{background:#343842;border-radius:999px}
  </style>
</head>
<body>
<div id="root"></div>
<script type="text/babel">
const {useState,useEffect}=React;

/* =========================
   1) SUPABASE CONFIG
   =========================
   Replace ONLY these two values with your Supabase Project URL
   and Publishable key (or legacy anon key). NEVER put service_role here.
*/
const SUPABASE_URL = 'https://nhybunfifvomilevobst.supabase.co/rest/v1/';
const SUPABASE_KEY = 'sb_publishable_TeSV8NnuxNz1XjxutbD1bQ_NJZ5JKe4';
const supabase = window.supabase.createClient(SUPABASE_URL,SUPABASE_KEY);

const THEME_PRESETS=[
  {name:'Violet (Défaut)',color:'#8B5CF6'},{name:'Rouge',color:'#EF4444'},{name:'Vert',color:'#22C55E'},
  {name:'Bleu',color:'#3B82F6'},{name:'Émeraude',color:'#10B981'},{name:'Or',color:'#F59E0B'},{name:'Rose',color:'#EC4899'}
];
const ICONS={video:'🎬',message:'💬',settings:'⚙️',palette:'🎨',card:'💳',send:'➤',users:'👥',dollar:'💶',plus:'＋',check:'✓',play:'▶',file:'📁',shield:'🛡️',brush:'🖌️',trash:'🗑️',link:'🔗',clock:'⏱️',external:'↗',youtube:'▶️',logout:'↪',login:'🔐'};
function Icon({name,size=20}){return <span style={{fontSize:size,lineHeight:1}}>{ICONS[name]||'•'}</span>}
function money(n){return `${Number(n||0).toFixed(0)}€`}
function normalizeVideoUrl(url){const t=url.trim();return /^https?:\/\//i.test(t)?t:`https://${t}`}
function getYouTubeId(url){
  try{const u=new URL(normalizeVideoUrl(url));const h=u.hostname.replace(/^www\./,'').toLowerCase();
    if(h==='youtu.be')return u.pathname.slice(1).split('/')[0]||null;
    if(['youtube.com','m.youtube.com'].includes(h)){
      if(u.pathname==='/watch')return u.searchParams.get('v');
      for(const p of ['/shorts/','/embed/','/live/'])if(u.pathname.startsWith(p))return u.pathname.split('/')[2]||null;
    }
  }catch{}
  return null;
}
function getYouTubeThumb(url){const id=getYouTubeId(url);return id?`https://img.youtube.com/vi/${id}/hqdefault.jpg`:''}

function AuthScreen({onDone}){
  const[mode,setMode]=useState('login');
  const[email,setEmail]=useState('');const[password,setPassword]=useState('');const[name,setName]=useState('');
  const[error,setError]=useState('');const[message,setMessage]=useState('');const[loading,setLoading]=useState(false);
  async function submit(e){e.preventDefault();setError('');setMessage('');setLoading(true);
    try{
      if(mode==='signup'){
        const{data,error}=await supabase.auth.signUp({email,password,options:{data:{display_name:name.trim()||email.split('@')[0]}}});
        if(error)throw error;
        if(!data.session)setMessage('Compte créé. Vérifie ton email puis connecte-toi.'); else onDone(data.session.user);
      }else{
        const{data,error}=await supabase.auth.signInWithPassword({email,password});if(error)throw error;onDone(data.user);
      }
    }catch(err){setError(err.message||'Une erreur est survenue.')}finally{setLoading(false)}
  }
  return <div className="min-h-screen bg-[#0f1115] flex items-center justify-center p-6 text-white">
    <div className="w-full max-w-md bg-gray-900 border border-gray-800 rounded-3xl p-7 shadow-2xl">
      <div className="text-center mb-7"><div className="mx-auto w-14 h-14 rounded-2xl flex items-center justify-center bg-violet-500/10 text-violet-400 mb-4"><Icon name="video" size={28}/></div><h1 className="text-3xl font-black">Wakzil <span className="text-violet-400">Studio</span></h1><p className="text-gray-400 mt-2">{mode==='login'?'Connecte-toi à ton espace':'Crée ton espace client'}</p></div>
      <form onSubmit={submit} className="space-y-4">
        {mode==='signup'&&<input value={name} onChange={e=>setName(e.target.value)} className="w-full bg-gray-800 border border-gray-700 rounded-xl p-3 outline-none text-white" placeholder="Nom / pseudo" />}
        <input required type="email" value={email} onChange={e=>setEmail(e.target.value)} className="w-full bg-gray-800 border border-gray-700 rounded-xl p-3 outline-none text-white" placeholder="Email" />
        <input required minLength="6" type="password" value={password} onChange={e=>setPassword(e.target.value)} className="w-full bg-gray-800 border border-gray-700 rounded-xl p-3 outline-none text-white" placeholder="Mot de passe" />
        {error&&<div className="p-3 rounded-xl bg-red-900/30 border border-red-800 text-red-300 text-sm">{error}</div>}
        {message&&<div className="p-3 rounded-xl bg-emerald-900/30 border border-emerald-800 text-emerald-300 text-sm">{message}</div>}
        <button disabled={loading} className="w-full py-3 rounded-xl font-bold text-white disabled:opacity-50 bg-violet-600 hover:bg-violet-500">{loading?'Chargement...':mode==='login'?'Se connecter':'Créer mon compte'}</button>
      </form>
      <button onClick={()=>{setMode(mode==='login'?'signup':'login');setError('');setMessage('')}} className="w-full mt-4 text-sm text-gray-400 hover:text-white">{mode==='login'?'Pas encore de compte ? Créer un compte':'Déjà un compte ? Se connecter'}</button>
    </div></div>
}

function App(){
 const[currentUser,setCurrentUser]=useState(null);const[profile,setProfile]=useState(null);const[loading,setLoading]=useState(true);
 const[activeTab,setActiveTab]=useState('portfolio');const[activeTicketId,setActiveTicketId]=useState(null);
 const[themeColor,setThemeColor]=useState('#8B5CF6');const[paypalLink,setPaypalLink]=useState('paypal.me/Wakzil');
 const[prices,setPrices]=useState({base:50,rush:20,minia:15,sousTitres:10});const[rushDeliveryTime,setRushDeliveryTime]=useState('48h');
 const[portfolio,setPortfolio]=useState([]);const[tickets,setTickets]=useState([]);const[activeTicket,setActiveTicket]=useState(null);const[msgInput,setMsgInput]=useState('');const[memberInput,setMemberInput]=useState('');
 const[orderError,setOrderError]=useState('');const[adminTab,setAdminTab]=useState('parametres');const[notice,setNotice]=useState('');
 const[newVideo,setNewVideo]=useState({url:'',title:'',category:'Mes vidéos',priceBase:50});const[videoError,setVideoError]=useState('');const[thumbPreview,setThumbPreview]=useState('');
 const[orderDraft,setOrderDraft]=useState({style:null,clientName:'',discordOrEmail:'',brief:'',link:'',options:{rush:false,minia:false,sousTitres:false}});

 const role=profile?.role||'client';
 const allPortfolio=portfolio;

 useEffect(()=>{
   let alive=true;
   supabase.auth.getSession().then(async({data})=>{if(!alive)return;if(data.session?.user)await loadUser(data.session.user);else setLoading(false)});
   const{data:{subscription}}=supabase.auth.onAuthStateChange(async(_,session)=>{if(session?.user)await loadUser(session.user);else{setCurrentUser(null);setProfile(null);setLoading(false)}});
   return()=>{alive=false;subscription.unsubscribe()};
 },[]);
 useEffect(()=>{if(currentUser)refreshPublicData()},[currentUser]);
 async function loadUser(user){setCurrentUser(user);const{data,error}=await supabase.from('profiles').select('*').eq('id',user.id).single();if(!error)setProfile(data);setLoading(false)}
 async function refreshPublicData(){
   const[settingsRes,portfolioRes]=await Promise.all([supabase.from('settings').select('*').eq('id',1).single(),supabase.from('portfolio').select('*').order('created_at',{ascending:true})]);
   if(settingsRes.data){setThemeColor(settingsRes.data.theme_color);setPaypalLink(settingsRes.data.paypal_link);setPrices({base:Number(settingsRes.data.base_price),rush:Number(settingsRes.data.rush_price),minia:Number(settingsRes.data.minia_price),sousTitres:Number(settingsRes.data.sous_titres_price)});setRushDeliveryTime(settingsRes.data.rush_delivery_time)}
   if(portfolioRes.data)setPortfolio(portfolioRes.data);
   if(currentUser)await loadTickets();
 }
 async function loadTickets(){const q=supabase.from('tickets').select('*').order('created_at',{ascending:false});const{data,error}=role==='client'?await q.eq('client_id',currentUser.id):await q;if(!error){setTickets(data||[]);if(activeTicketId&&!data?.some(t=>t.id===activeTicketId))setActiveTicketId(null)}}
 async function openTicket(ticket){setActiveTicketId(ticket.id);const{data}=await supabase.from('ticket_messages').select('*').eq('ticket_id',ticket.id).order('created_at',{ascending:true});setActiveTicket({...ticket,messages:data||[]})}
 useEffect(()=>{if(activeTicketId){const t=tickets.find(x=>x.id===activeTicketId);if(t)openTicket(t)}else setActiveTicket(null)},[activeTicketId,tickets.length]);
 function goTab(tab){if(tab==='admin'&&role!=='admin')return;if(tab==='tickets')loadTickets();setActiveTab(tab)}
 function calculatePrice(){let total=orderDraft.style?Number(orderDraft.style.price_base):Number(prices.base);if(orderDraft.options.rush)total+=Number(prices.rush);if(orderDraft.options.minia)total+=Number(prices.minia);if(orderDraft.options.sousTitres)total+=Number(prices.sousTitres);return total}
 async function handleCreateTicket(){setOrderError('');if(!orderDraft.clientName.trim()||!orderDraft.discordOrEmail.trim()||!orderDraft.brief.trim()||!orderDraft.link.trim()){setOrderError('Veuillez remplir tous les champs obligatoires.');return}
   const code=`#TICK-${Math.floor(Math.random()*100000).toString().padStart(5,'0')}`;const payload={public_code:code,title:orderDraft.style?`Commande: ${orderDraft.style.title}`:'Nouvelle Commande',price:calculatePrice(),client_id:currentUser.id,client_name:orderDraft.clientName.trim(),contact:orderDraft.discordOrEmail.trim(),brief:orderDraft.brief.trim(),rush:orderDraft.options.rush,rush_delivery_time:orderDraft.options.rush?rushDeliveryTime:null,source_link:orderDraft.link.trim(),selected_style_id:orderDraft.style?.id||null};
   const{data,error}=await supabase.from('tickets').insert(payload).select('*').single();if(error){setOrderError(error.message);return}
   const{text:errorText}=await insertMessage(data.id,`Commande créée.\nBrief : ${payload.brief}\n\nLien des rushs : ${payload.source_link}${payload.rush?`\n\nLivraison Express : ${payload.rush_delivery_time}`:''}`);if(errorText){setOrderError(errorText);return}
   setTickets(t=>[data,...t]);setOrderDraft({style:null,clientName:'',discordOrEmail:'',brief:'',link:'',options:{rush:false,minia:false,sousTitres:false}});setActiveTicketId(data.id);setActiveTab('tickets');
 }
 async function insertMessage(ticketId,textValue){const{error}=await supabase.from('ticket_messages').insert({ticket_id:ticketId,sender_id:currentUser.id,text:textValue});return {text:error?.message||null}}
 async function sendMessage(){if(!msgInput.trim()||!activeTicket)return;const{error}=await supabase.from('ticket_messages').insert({ticket_id:activeTicket.id,sender_id:currentUser.id,text:msgInput.trim()});if(error){setNotice(error.message);return}setMsgInput('');await openTicket(activeTicket)}
 async function addMemberToTicket(){if(!memberInput.trim()||!activeTicket)return;await insertMessage(activeTicket.id,`🤝 ${memberInput.trim()} a été ajouté au ticket.`);setMemberInput('');await openTicket(activeTicket)}
 async function saveSettings(patch){const{error}=await supabase.from('settings').update(patch).eq('id',1);if(error)setNotice(error.message);else{setNotice('Paramètres enregistrés.');await refreshPublicData()}}
 async function addVideo(e){e.preventDefault();setVideoError('');const url=normalizeVideoUrl(newVideo.url);const id=getYouTubeId(url);if(!id){setVideoError('Pour la miniature automatique, utilise un lien YouTube (youtube.com ou youtu.be).');return}if(!newVideo.title.trim()){setVideoError('Ajoute un titre.');return}
   const{data,error}=await supabase.from('portfolio').insert({title:newVideo.title.trim(),category:newVideo.category.trim()||'Mes vidéos',price_base:Number(newVideo.priceBase)||50,video_url:url,thumbnail_url:`https://img.youtube.com/vi/${id}/hqdefault.jpg`,is_custom:true,created_by:currentUser.id}).select('*').single();if(error){setVideoError(error.message);return}setPortfolio(p=>[...p,data]);setNewVideo({url:'',title:'',category:'Mes vidéos',priceBase:50});setThumbPreview('');setNotice('Exemple ajouté.');}
 async function deleteVideo(id){if(!confirm('Supprimer cet exemple ?'))return;const{error}=await supabase.from('portfolio').delete().eq('id',id);if(error)setNotice(error.message);else{setPortfolio(p=>p.filter(v=>v.id!==id));if(activeTab==='portfolio')setNotice('Exemple supprimé.')}}
 async function signOut(){await supabase.auth.signOut();}
 if(loading)return <div className="min-h-screen bg-[#0f1115] text-white flex items-center justify-center">Chargement...</div>;
 if(!currentUser||!profile)return <AuthScreen onDone={u=>loadUser(u)} />;
 const clickablePaypalLink=()=>paypalLink.trim()?( /^https?:\/\//i.test(paypalLink.trim())?paypalLink.trim():`https://${paypalLink.trim()}` ):'#';
 const tabs=[{id:'portfolio',label:'Exemples',icon:'file'},{id:'order',label:'Commander',icon:'plus'},{id:'tickets',label:'Tickets',icon:'message'}];if(role==='admin')tabs.push({id:'admin',label:'Admin',icon:'settings'});
 return <div className="min-h-screen bg-[#0f1115] font-sans text-white flex flex-col overflow-hidden">
   <header className="bg-gray-900 border-b border-gray-800 p-4 flex justify-between items-center sticky top-0 z-50">
     <div className="flex items-center gap-3"><div className="p-2 rounded-lg" style={{backgroundColor:`${themeColor}20`,color:themeColor}}><Icon name="video" size={28}/></div><h1 className="text-2xl font-bold tracking-tight">Wakzil <span style={{color:themeColor}}>Studio</span></h1></div>
     <div className="flex items-center gap-3"><div className="hidden sm:block text-right"><div className="text-sm font-bold">{profile.display_name||currentUser.email}</div><div className="text-xs text-gray-500 capitalize">{role}</div></div><button onClick={signOut} className="p-2.5 rounded-xl bg-gray-800 hover:bg-gray-700" title="Déconnexion"><Icon name="logout" size={18}/></button></div>
   </header>
   <div className="flex flex-1 overflow-hidden">
     <nav className="bg-gray-900 w-64 border-r border-gray-800 p-4 flex flex-col gap-2 h-[calc(100vh-73px)] sticky top-[73px]">
       {tabs.map(tab=><button key={tab.id} onClick={()=>goTab(tab.id)} className={`flex items-center gap-3 px-4 py-3 rounded-xl transition-all ${activeTab===tab.id?'text-white':'text-gray-400 hover:bg-gray-800 hover:text-white'}`} style={activeTab===tab.id?{backgroundColor:`${themeColor}20`,color:themeColor,borderRight:`4px solid ${themeColor}`}:{}}><Icon name={tab.icon} size={20}/><span className="font-medium">{tab.label}</span></button>)}
     </nav>
     <main className="flex-1 overflow-y-auto bg-black/20">
       {activeTab==='portfolio'&&<div className="p-8 space-y-6"><div className="flex flex-wrap justify-between gap-4 items-center"><div><h2 className="text-3xl font-bold">Nos Exemples de Montage</h2><p className="text-gray-400 mt-1">Clique sur une vidéo pour voir ton exemple.</p></div>{role==='admin'&&<button onClick={()=>{setAdminTab('exemples');setActiveTab('admin')}} className="px-4 py-2 rounded-xl font-semibold hover:brightness-110" style={{backgroundColor:themeColor}}>Gérer les exemples</button>}</div>
         <div className="grid grid-cols-1 md:grid-cols-2 gap-6">{allPortfolio.map(item=><div key={item.id} className="bg-gray-800 rounded-2xl overflow-hidden border border-gray-700 hover:border-gray-600 transition-all group"><a href={item.video_url} target="_blank" rel="noreferrer" className="relative block h-48 bg-gray-900 overflow-hidden"><img src={item.thumbnail_url} alt={item.title} className="w-full h-full object-cover opacity-70 group-hover:opacity-90 transition-opacity"/><div className="absolute inset-0 flex items-center justify-center"><div className="w-14 h-14 rounded-full bg-black/60 backdrop-blur flex items-center justify-center"><Icon name="play" size={26}/></div></div><span className="absolute top-4 left-4 px-3 py-1 rounded-full text-xs font-bold bg-black/60 backdrop-blur">{item.category}</span></a><div className="p-5"><h3 className="text-xl font-bold">{item.title}</h3><p className="text-gray-400 text-sm mt-1">À partir de {money(item.price_base)}</p><button onClick={()=>{setOrderDraft(o=>({...o,style:item}));setActiveTab('order')}} className="w-full mt-4 py-2.5 rounded-lg font-medium hover:brightness-110" style={{backgroundColor:themeColor}}>Utiliser ce style</button>{role==='admin'&&item.is_custom&&<button onClick={()=>deleteVideo(item.id)} className="w-full mt-2 py-2 rounded-lg bg-red-900/30 border border-red-800 text-red-300">Supprimer</button>}</div></div>)}</div>
       </div>}

       {activeTab==='order'&&<div className="p-8 max-w-5xl mx-auto space-y-8"><div className="text-center"><h2 className="text-3xl font-bold">Nouvelle Commande</h2><p className="text-gray-400 mt-2">Ton compte permet de retrouver ton ticket sur tous tes appareils.</p></div><div className="grid grid-cols-1 lg:grid-cols-3 gap-8"><div className="lg:col-span-2 space-y-6 bg-gray-800 p-6 rounded-2xl border border-gray-700">
          {orderDraft.style&&<div className="flex items-center justify-between p-4 bg-gray-900 rounded-xl border border-gray-700"><div><p className="text-sm text-gray-400">Style sélectionné</p><p className="font-bold">{orderDraft.style.title}</p></div><button onClick={()=>setOrderDraft(o=>({...o,style:null}))} className="text-sm text-red-400">Retirer</button></div>}
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4"><input value={orderDraft.clientName} onChange={e=>setOrderDraft(o=>({...o,clientName:e.target.value}))} className="w-full bg-gray-900 border border-gray-700 rounded-xl p-3 outline-none" placeholder="Votre pseudo *"/><input value={orderDraft.discordOrEmail} onChange={e=>setOrderDraft(o=>({...o,discordOrEmail:e.target.value}))} className="w-full bg-gray-900 border border-gray-700 rounded-xl p-3 outline-none" placeholder="Discord ou Email *"/></div>
          <textarea value={orderDraft.brief} onChange={e=>setOrderDraft(o=>({...o,brief:e.target.value}))} className="w-full bg-gray-900 border border-gray-700 rounded-xl p-4 outline-none h-36 resize-none" placeholder="Brief / explications du montage *"/>
          <input value={orderDraft.link} onChange={e=>setOrderDraft(o=>({...o,link:e.target.value}))} className="w-full bg-gray-900 border border-gray-700 rounded-xl p-4 outline-none" placeholder="Lien des rushs (Google Drive, WeTransfer...) *"/>
          <div><div className="text-sm font-medium text-gray-300 mb-3">Options supplémentaires</div><div className="space-y-2">{[{id:'rush',label:`Livraison Express (${rushDeliveryTime})`,price:prices.rush},{id:'minia',label:'Création Miniature',price:prices.minia},{id:'sousTitres',label:'Sous-titres animés',price:prices.sousTitres}].map(opt=><label key={opt.id} className="flex items-center justify-between p-4 rounded-xl border border-gray-700 bg-gray-900 cursor-pointer"><span className="flex items-center gap-3"><input type="checkbox" checked={orderDraft.options[opt.id]} onChange={e=>setOrderDraft(o=>({...o,options:{...o.options,[opt.id]:e.target.checked}}))} className="w-5 h-5"/><span className="font-medium">{opt.label}</span></span><span className="text-gray-400">+{money(opt.price)}</span></label>)}</div></div>
        </div><div><div className="bg-gray-800 p-6 rounded-2xl border border-gray-700 sticky top-6"><h3 className="text-xl font-bold mb-5">Résumé</h3><div className="space-y-3 text-sm text-gray-300"><div className="flex justify-between"><span>Base</span><span>{money(orderDraft.style?orderDraft.style.price_base:prices.base)}</span></div>{orderDraft.options.rush&&<div className="flex justify-between"><span>Express ({rushDeliveryTime})</span><span>+{money(prices.rush)}</span></div>}{orderDraft.options.minia&&<div className="flex justify-between"><span>Miniature</span><span>+{money(prices.minia)}</span></div>}{orderDraft.options.sousTitres&&<div className="flex justify-between"><span>Sous-titres</span><span>+{money(prices.sousTitres)}</span></div>}</div><div className="border-t border-gray-700 mt-5 pt-5"><div className="flex justify-between items-end mb-5"><span className="text-gray-400">Total estimé</span><span className="text-3xl font-bold">{money(calculatePrice())}</span></div>{orderError&&<div className="mb-4 p-3 rounded-xl bg-red-900/40 border border-red-800 text-red-300 text-sm">{orderError}</div>}<button onClick={handleCreateTicket} className="w-full py-3.5 rounded-xl font-bold text-lg hover:brightness-110" style={{backgroundColor:themeColor}}>Ouvrir un ticket</button></div></div></div></div></div>}

       {activeTab==='tickets'&&<div className="flex h-full">{tickets.length===0?<div className="flex-1 flex flex-col items-center justify-center text-gray-400 gap-4"><Icon name="message" size={54}/><p>Aucun ticket.</p><button onClick={()=>setActiveTab('order')} style={{color:themeColor}}>Créer une commande</button></div>:<><div className="w-80 border-r border-gray-800 bg-gray-900 overflow-y-auto"><div className="p-4 border-b border-gray-800"><h3 className="text-lg font-bold">{role==='client'?'Vos Tickets':'Tickets'}</h3></div><div className="p-2 space-y-2">{tickets.map(t=><button key={t.id} onClick={()=>openTicket(t)} className={`w-full text-left p-4 rounded-xl ${activeTicketId===t.id?'bg-gray-800':'hover:bg-gray-800/50'}`} style={activeTicketId===t.id?{borderLeft:`4px solid ${themeColor}`}:{borderLeft:'4px solid transparent'}}><div className="flex justify-between items-center gap-2 mb-2"><span className="font-bold truncate">{t.public_code}</span><span className="text-xs px-2 py-1 bg-gray-700 rounded-full">{t.status}</span></div><p className="text-sm text-gray-400 truncate">{t.client_name} - {t.title}</p></button>)}</div></div><div className="flex-1 flex flex-col bg-gray-800">{activeTicket?<><div className="p-4 border-b border-gray-700 bg-gray-900 flex flex-wrap justify-between gap-3 items-center"><div><h3 className="text-xl font-bold">{activeTicket.title} <span className="text-sm font-normal text-gray-400">{activeTicket.public_code}</span></h3><p className="text-sm text-gray-400">Client : <span className="text-white">{activeTicket.client_name}</span> ({activeTicket.contact})</p></div><div className="flex items-center gap-2">{role!=='client'&&<div className="flex bg-gray-800 rounded-lg border border-gray-700"><input value={memberInput} onChange={e=>setMemberInput(e.target.value)} placeholder="Ajouter un pote..." className="bg-transparent px-2 text-xs outline-none"/><button onClick={addMemberToTicket} className="p-1 bg-gray-700 rounded"><Icon name="plus" size={16}/></button></div>}<button onClick={()=>window.open(clickablePaypalLink(),'_blank')} className="px-4 py-2 rounded-lg font-bold hover:brightness-110" style={{backgroundColor:themeColor}}><Icon name="card" size={17}/> Payer {money(activeTicket.price)}</button></div></div><div className="flex-1 overflow-y-auto p-6 space-y-4">{(activeTicket.messages||[]).map(msg=><div key={msg.id} className={`flex ${msg.sender_id===currentUser.id?'justify-end':'justify-start'}`}><div className={`max-w-[75%] rounded-2xl p-4 ${msg.sender_id===currentUser.id?'text-white':'bg-gray-700 text-white'}`} style={msg.sender_id===currentUser.id?{backgroundColor:themeColor}:{}}><div className="text-xs opacity-70 mb-1">{msg.sender_id===currentUser.id?'Vous':role==='admin'?'Membre de l’équipe':'Monteur'}</div><div className="whitespace-pre-wrap">{msg.text}</div></div></div>)}</div><div className="p-4 bg-gray-900 border-t border-gray-700"><div className="flex gap-2"><input value={msgInput} onChange={e=>setMsgInput(e.target.value)} onKeyDown={e=>e.key==='Enter'&&sendMessage()} placeholder="Écrivez un message..." className="flex-1 bg-gray-800 border border-gray-700 rounded-xl px-4 py-3 outline-none"/><button onClick={sendMessage} className="p-3 rounded-xl" style={{backgroundColor:themeColor}}><Icon name="send" size={20}/></button></div></div></>:<div className="flex-1 flex items-center justify-center text-gray-500">Sélectionnez un ticket.</div>}</div></>}</div>}

       {activeTab==='admin'&&role==='admin'&&<div className="p-8 max-w-6xl mx-auto space-y-6"><div><h2 className="text-3xl font-bold">Panneau d’Administration</h2><p className="text-gray-400 mt-1">Ces réglages sont stockés dans Supabase et sont donc partagés avec tous les visiteurs.</p></div><div className="flex gap-3 border-b border-gray-800 pb-2 overflow-x-auto">{[{id:'parametres',label:'Paramètres'},{id:'tarifs',label:'Tarifs'},{id:'exemples',label:'Exemples'},{id:'equipe',label:'Équipe'}].map(t=><button key={t.id} onClick={()=>setAdminTab(t.id)} className={`px-4 py-2 ${adminTab===t.id?'text-white':'text-gray-500'}`} style={adminTab===t.id?{borderBottom:`2px solid ${themeColor}`,color:themeColor}: {}}>{t.label}</button>)}</div>
         {notice&&<div className="p-3 rounded-xl bg-emerald-900/30 border border-emerald-800 text-emerald-300">{notice}</div>}
         {adminTab==='parametres'&&<div className="space-y-6"><div className="bg-gray-800 rounded-2xl p-6 border border-gray-700"><h3 className="text-xl font-bold mb-4">Thème</h3><div className="flex flex-wrap gap-3 mb-5">{THEME_PRESETS.map(p=><button key={p.color} onClick={()=>{setThemeColor(p.color);saveSettings({theme_color:p.color})}} title={p.name} className="w-10 h-10 rounded-full border-2" style={{backgroundColor:p.color,borderColor:themeColor===p.color?'white':'transparent'}}>{themeColor===p.color&&<Icon name="check" size={16}/>}</button>)}</div><div className="flex items-center gap-3"><input type="color" value={themeColor} onChange={e=>{setThemeColor(e.target.value);saveSettings({theme_color:e.target.value})}}/><span className="font-mono text-sm text-gray-400">{themeColor.toUpperCase()}</span></div></div><div className="bg-gray-800 rounded-2xl p-6 border border-gray-700"><h3 className="text-xl font-bold mb-4">Paiement</h3><input value={paypalLink} onChange={e=>setPaypalLink(e.target.value)} onBlur={()=>saveSettings({paypal_link:paypalLink})} className="w-full max-w-lg bg-gray-900 border border-gray-700 rounded-xl p-3 outline-none" placeholder="paypal.me/VotrePseudo"/></div></div>}
         {adminTab==='tarifs'&&<div className="bg-gray-800 rounded-2xl p-6 border border-gray-700"><h3 className="text-xl font-bold mb-5">Prix et livraison Express</h3><div className="grid grid-cols-1 md:grid-cols-2 gap-4 max-w-2xl">{[['base','Montage de base','base_price'],['rush','Prix Express','rush_price'],['minia','Miniature','minia_price'],['sousTitres','Sous-titres','sous_titres_price']].map(([key,label,col])=><label key={key} className="bg-gray-900 p-4 rounded-xl"><span className="block text-sm text-gray-400 mb-2">{label}</span><div className="flex gap-2"><input type="number" min="0" value={prices[key]} onChange={e=>setPrices(p=>({...p,[key]:Number(e.target.value)}))} onBlur={()=>saveSettings({[col]:prices[key]})} className="w-full bg-gray-800 border border-gray-700 rounded-lg p-2 outline-none"/><span className="self-center text-gray-400">€</span></div></label>)}<label className="bg-gray-900 p-4 rounded-xl md:col-span-2"><span className="block text-sm text-gray-400 mb-2">Délai de livraison Express</span><input value={rushDeliveryTime} onChange={e=>setRushDeliveryTime(e.target.value)} onBlur={()=>saveSettings({rush_delivery_time:rushDeliveryTime})} className="w-full bg-gray-800 border border-gray-700 rounded-lg p-2 outline-none" placeholder="Ex : 24h, 48h, 72h"/></label></div></div>}
         {adminTab==='exemples'&&<div className="space-y-6"><div className="bg-gray-800 rounded-2xl p-6 border border-gray-700"><h3 className="text-xl font-bold mb-4">Ajouter une vidéo</h3><form onSubmit={addVideo} className="grid grid-cols-1 md:grid-cols-2 gap-4"><input required value={newVideo.title} onChange={e=>setNewVideo(v=>({...v,title:e.target.value}))} className="bg-gray-900 border border-gray-700 rounded-xl p-3 outline-none" placeholder="Titre"/><input value={newVideo.category} onChange={e=>setNewVideo(v=>({...v,category:e.target.value}))} className="bg-gray-900 border border-gray-700 rounded-xl p-3 outline-none" placeholder="Catégorie"/><input required value={newVideo.url} onChange={e=>{const value=e.target.value;setNewVideo(v=>({...v,url:value}));setThumbPreview(getYouTubeThumb(value))}} className="md:col-span-2 bg-gray-900 border border-gray-700 rounded-xl p-3 outline-none" placeholder="Lien YouTube"/><div className="flex gap-2 items-center"><input type="number" min="0" value={newVideo.priceBase} onChange={e=>setNewVideo(v=>({...v,priceBase:Number(e.target.value)}))} className="w-32 bg-gray-900 border border-gray-700 rounded-xl p-3 outline-none"/><span className="text-gray-400">€ prix de base</span></div>{thumbPreview&&<img src={thumbPreview} alt="Aperçu miniature" className="h-24 rounded-xl object-cover"/>}{videoError&&<div className="md:col-span-2 p-3 rounded-xl bg-red-900/30 border border-red-800 text-red-300 text-sm">{videoError}</div>}<button className="md:col-span-2 py-3 rounded-xl font-bold hover:brightness-110" style={{backgroundColor:themeColor}}>Ajouter l’exemple</button></form></div><div className="grid grid-cols-1 md:grid-cols-2 gap-5">{portfolio.filter(v=>v.is_custom).map(v=><div key={v.id} className="bg-gray-800 rounded-2xl overflow-hidden border border-gray-700"><img src={v.thumbnail_url} className="w-full h-44 object-cover"/><div className="p-4"><div className="font-bold">{v.title}</div><div className="text-sm text-gray-400">{v.category} · {money(v.price_base)}</div><button onClick={()=>deleteVideo(v.id)} className="mt-3 w-full py-2 rounded-xl bg-red-900/30 border border-red-800 text-red-300">Supprimer</button></div></div>)}</div></div>}
         {adminTab==='equipe'&&<div className="bg-gray-800 rounded-2xl p-6 border border-gray-700"><h3 className="text-xl font-bold mb-5">Comptes autorisés</h3><p className="text-gray-400 text-sm">Les rôles sont gérés dans <b>Supabase → Table Editor → profiles</b>. Mets <b>admin</b> pour ton compte et <b>monteur</b> pour un monteur. Ne mets jamais une clé service_role dans ce site.</p></div>}
       </div>}
       {activeTab==='admin'&&role!=='admin'&&<div className="p-8 text-red-300 flex items-center gap-2"><Icon name="shield"/> Accès refusé.</div>}
     </main>
   </div>
 </div>
}
ReactDOM.createRoot(document.getElementById('root')).render(<App/>);
</script>
</body></html>
