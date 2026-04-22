{import { useState, useRef, useEffect } from "react";

const UDJ = {
  maroon: "#6B1F2A",
  maroonDark: "#4A1219",
  maroonLight: "#F9F0F1",
  maroonMid: "#8B2635",
  white:  "#FFFFFF",
  lgray:  "#F4F4F4",
  border: "#D8C8CA",
  text:   "#2C0A0E",
};

const POSITIONS = ["LW","C","RW","LD","RD","G"];
const HANDS = ["L","R"];
const TABS = ["Even Strength","Power Play","Penalty Kill","Extra Attacker","Print Summary"];

const LINE_COLORS = [
  {name:"Red",    bg:"#FCEBEB", border:"#E24B4A", text:"#A32D2D"},
  {name:"Blue",   bg:"#E6F1FB", border:"#378ADD", text:"#185FA5"},
  {name:"Green",  bg:"#EAF3DE", border:"#639922", text:"#3B6D11"},
  {name:"Amber",  bg:"#FAEEDA", border:"#EF9F27", text:"#854F0B"},
  {name:"Purple", bg:"#EEEDFE", border:"#7F77DD", text:"#534AB7"},
  {name:"Teal",   bg:"#E1F5EE", border:"#1D9E75", text:"#0F6E56"},
  {name:"Pink",   bg:"#FBEAF0", border:"#D4537E", text:"#993556"},
  {name:"White",  bg:"#FFFFFF", border:"#C8C8C8", text:"#5F5E5A"},
  {name:"Gray",   bg:"#E0DFDA", border:"#888780", text:"#444441"},
  {name:"Black",  bg:"#C2C1BE", border:"#2C2C2A", text:"#2C2C2A"},
];

const posBg    = {LW:"#E6F1FB",C:"#EAF3DE",RW:"#E6F1FB",LD:"#FBEAF0",RD:"#FBEAF0",G:"#FAEEDA"};
const posColor = {LW:"#185FA5",C:"#3B6D11",RW:"#185FA5",LD:"#993556",RD:"#993556",G:"#854F0B"};

const mkLine = (label) => ({lw:null,c:null,rw:null,label,color:null,notes:""});
const mkPair = (label) => ({ld:null,rd:null,label,color:null,notes:""});
const mkUnit = (size,label) => ({slots:Array(size).fill(null),label,color:null,positions:null,notes:""});

const DEFAULT_PP_POS = [{x:50,y:20},{x:15,y:50},{x:50,y:60},{x:85,y:50},{x:50,y:85}];
const OZ_W=300, OZ_H=380;

function OffensiveZone() {
  const W=OZ_W,H=OZ_H,netW=58,netH=14,netLeft=(W-netW)/2,netY=3,creaseR=52,goalLineY=52,blueLineY=H-20,dotR=13;
  const dots=[{cx:W*0.22,cy:H*0.38},{cx:W*0.78,cy:H*0.38}];
  return (
    <svg width="100%" height="100%" viewBox={`0 0 ${W} ${H}`} style={{position:"absolute",inset:0,pointerEvents:"none"}}>
      <rect width={W} height={H} rx={10} fill="#d9eef7"/>
      <rect x={1} y={1} width={W-2} height={H-2} rx={10} fill="none" stroke="#7bbbd4" strokeWidth="3"/>
      <line x1={0} y1={blueLineY} x2={W} y2={blueLineY} stroke="#1a55cc" strokeWidth="5"/>
      <line x1={6} y1={goalLineY} x2={W-6} y2={goalLineY} stroke="#cc2222" strokeWidth="2"/>
      <path d={`M ${W/2-netW/2} ${goalLineY} A ${creaseR} ${creaseR} 0 0 1 ${W/2+netW/2} ${goalLineY}`} fill="#b3d9f0" stroke="#7bbbd4" strokeWidth="1.5"/>
      <rect x={netLeft} y={netY} width={netW} height={netH} fill="#ccc" stroke="#555" strokeWidth="1.5" rx={2}/>
      <line x1={netLeft} y1={netY} x2={netLeft-6} y2={2} stroke="#999" strokeWidth="1"/>
      <line x1={netLeft+netW} y1={netY} x2={netLeft+netW+6} y2={2} stroke="#999" strokeWidth="1"/>
      <line x1={netLeft-6} y1={2} x2={netLeft+netW+6} y2={2} stroke="#999" strokeWidth="1"/>
      {dots.map((d,i)=>(
        <g key={i}>
          <circle cx={d.cx} cy={d.cy} r={dotR} fill="none" stroke="#cc2222" strokeWidth="2"/>
          <circle cx={d.cx} cy={d.cy} r={3} fill="#cc2222"/>
          <line x1={d.cx-dotR-7} y1={d.cy} x2={d.cx-dotR+2} y2={d.cy} stroke="#cc2222" strokeWidth="1.5"/>
          <line x1={d.cx+dotR-2} y1={d.cy} x2={d.cx+dotR+7} y2={d.cy} stroke="#cc2222" strokeWidth="1.5"/>
          <line x1={d.cx} y1={d.cy-dotR-7} x2={d.cx} y2={d.cy-dotR+2} stroke="#cc2222" strokeWidth="1.5"/>
          <line x1={d.cx} y1={d.cy+dotR-2} x2={d.cx} y2={d.cy+dotR+7} stroke="#cc2222" strokeWidth="1.5"/>
        </g>
      ))}
      <circle cx={W/2} cy={H*0.6} r={3} fill="#cc2222"/>
    </svg>
  );
}

function ColorPicker({ value, onChange }) {
  const [open,setOpen]=useState(false);
  const cur=LINE_COLORS.find(c=>c.name===value)||null;
  return (
    <div style={{position:"relative",display:"inline-block"}}>
      <button onClick={()=>setOpen(o=>!o)} style={{width:22,height:22,borderRadius:"50%",cursor:"pointer",border:`1px solid ${UDJ.border}`,background:cur?cur.bg:UDJ.white,display:"flex",alignItems:"center",justifyContent:"center",padding:0}}>
        {cur?<div style={{width:12,height:12,borderRadius:"50%",background:cur.border}}/>:<span style={{fontSize:10,color:"#aaa"}}>+</span>}
      </button>
      {open&&(
        <div style={{position:"absolute",top:26,left:0,zIndex:99,background:UDJ.white,border:`1px solid ${UDJ.border}`,borderRadius:8,padding:6,display:"flex",flexWrap:"wrap",gap:5,width:112,boxShadow:"0 4px 12px rgba(0,0,0,0.12)"}}>
          <button onClick={()=>{onChange(null);setOpen(false);}} style={{width:20,height:20,borderRadius:"50%",cursor:"pointer",border:`1px solid ${UDJ.border}`,background:UDJ.white,fontSize:9,color:"#aaa",padding:0}}>✕</button>
          {LINE_COLORS.map(c=>(
            <button key={c.name} title={c.name} onClick={()=>{onChange(c.name);setOpen(false);}} style={{width:20,height:20,borderRadius:"50%",cursor:"pointer",background:c.bg,border:`2px solid ${c.border}`,padding:0}}/>
          ))}
        </div>
      )}
    </div>
  );
}

function LineHeader({ label,setLabel,color,setColor,notes,setNotes,onRemove,canRemove }) {
  const [editing,setEditing]=useState(false);
  const [draft,setDraft]=useState(label);
  const [showNotes,setShowNotes]=useState(false);
  const cur=LINE_COLORS.find(c=>c.name===color)||null;
  return (
    <div style={{marginBottom:8}}>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:showNotes?6:0}}>
        <div style={{display:"flex",alignItems:"center",gap:6}}>
          <ColorPicker value={color} onChange={setColor}/>
          {editing?(
            <input autoFocus value={draft} onChange={e=>setDraft(e.target.value)}
              onBlur={()=>{setLabel(draft||label);setEditing(false);}}
              onKeyDown={e=>{if(e.key==="Enter"){setLabel(draft||label);setEditing(false);}}}
              style={{fontSize:13,fontWeight:600,padding:"1px 6px",borderRadius:5,border:`1px solid ${UDJ.border}`,width:130,color:UDJ.text}}
            />
          ):(
            <span onClick={()=>{setDraft(label);setEditing(true);}} title="Click to rename"
              style={{fontSize:13,fontWeight:600,cursor:"text",padding:"2px 7px",borderRadius:5,color:cur?cur.text:UDJ.maroon,background:cur?cur.bg:"transparent",border:cur?`1px solid ${cur.border}`:"1px solid transparent"}}>
              {label} <span style={{fontSize:10,opacity:0.35}}>✎</span>
            </span>
          )}
          <button onClick={()=>setShowNotes(n=>!n)} style={{fontSize:10,padding:"1px 7px",borderRadius:5,cursor:"pointer",border:`1px solid ${UDJ.border}`,background:notes?"#FFF0F0":"transparent",color:notes?UDJ.maroon:"#888",fontWeight:notes?700:400}}>
            {notes?"📋":"+ note"}
          </button>
        </div>
        {canRemove&&<button onClick={onRemove} style={{fontSize:11,padding:"2px 8px",borderRadius:5,cursor:"pointer",border:`1px solid ${UDJ.border}`,color:UDJ.maroon,background:"transparent"}}>Remove</button>}
      </div>
      {showNotes&&(
        <input value={notes} onChange={e=>setNotes(e.target.value)} placeholder="Line notes (e.g. defensive zone starts, aggressive forecheck...)"
          style={{width:"100%",padding:"4px 8px",fontSize:12,borderRadius:6,border:`1px solid ${UDJ.maroon}`,background:"#FFF8F8",color:UDJ.text,boxSizing:"border-box"}}/>
      )}
    </div>
  );
}

function PlayerBadge({ player, scratch }) {
  if (!player) return null;
  return (
    <div style={{display:"flex",alignItems:"center",gap:4,flexWrap:"wrap",opacity:scratch?0.5:1}}>
      <span style={{fontWeight:500,fontSize:12,color:UDJ.text,textDecoration:scratch?"line-through":"none"}}>#{player.number} {player.name}</span>
      <span style={{fontSize:11,padding:"1px 5px",borderRadius:4,background:posBg[player.pos],color:posColor[player.pos],fontWeight:500}}>{player.pos}</span>
      <span style={{fontSize:11,padding:"1px 5px",borderRadius:4,background:UDJ.lgray,color:"#555",border:`1px solid ${UDJ.border}`}}>{player.hand}</span>
    </div>
  );
}

function Slot({ label,player,onDrop,onClick,selected }) {
  const [over,setOver]=useState(false);
  return (
    <div onClick={onClick}
      onDragOver={e=>{e.preventDefault();setOver(true);}}
      onDragLeave={()=>setOver(false)}
      onDrop={e=>{e.preventDefault();setOver(false);onDrop(e);}}
      style={{minHeight:52,padding:"6px 10px",borderRadius:8,cursor:"pointer",border:selected?`2px solid ${UDJ.maroon}`:over?`1.5px dashed ${UDJ.maroon}`:`1px solid ${UDJ.border}`,background:selected?UDJ.maroonLight:over?"#FDF5F5":UDJ.white,display:"flex",flexDirection:"column",justifyContent:"center",gap:2}}>
      <span style={{fontSize:10,color:"#888",fontWeight:600,textTransform:"uppercase",letterSpacing:"0.06em"}}>{label}</span>
      {player?<PlayerBadge player={player}/>:<span style={{fontSize:12,color:"#ccc"}}>Empty</span>}
    </div>
  );
}

function RosterCard({ player,onDrag,onClick,selected,assigned,scratchStatus }) {
  const isScratch = scratchStatus==="scratch";
  const isInjured = scratchStatus==="injured";
  return (
    <div draggable onDragStart={onDrag} onClick={onClick}
      style={{padding:"7px 10px",borderRadius:8,cursor:"pointer",
        border:selected?`2px solid ${UDJ.maroon}`:`1px solid ${UDJ.border}`,
        background:isScratch?"#FFF0F0":isInjured?"#FFFBE6":assigned?UDJ.lgray:selected?UDJ.maroonLight:UDJ.white,
        opacity:assigned&&!isScratch&&!isInjured?0.55:1,display:"flex",flexDirection:"column",gap:2}}>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start"}}>
        <PlayerBadge player={player}/>
        {(isScratch||isInjured)&&(
          <span style={{fontSize:9,fontWeight:700,padding:"1px 5px",borderRadius:4,background:isScratch?UDJ.maroon:"#E6A800",color:"#fff",marginLeft:4,flexShrink:0}}>
            {isScratch?"SCR":"INJ"}
          </span>
        )}
      </div>
      {assigned&&!isScratch&&!isInjured&&<span style={{fontSize:10,color:"#aaa"}}>assigned</span>}
    </div>
  );
}

function PPUnit({ unit,ui,setUnits,sectionBg,SlotEl }) {
  const positions=(unit.positions&&unit.positions.length===5)?unit.positions:DEFAULT_PP_POS.map(p=>({...p}));
  const rinkRef=useRef(null);
  const dragging=useRef(null);
  const startDrag=(e,si)=>{
    e.preventDefault();e.stopPropagation();dragging.current=si;
    const onMove=(ev)=>{
      if(dragging.current===null||!rinkRef.current)return;
      const rect=rinkRef.current.getBoundingClientRect();
      const cx=ev.touches?ev.touches[0].clientX:ev.clientX;
      const cy=ev.touches?ev.touches[0].clientY:ev.clientY;
      const x=Math.min(92,Math.max(8,((cx-rect.left)/rect.width)*100));
      const y=Math.min(92,Math.max(8,((cy-rect.top)/rect.height)*100));
      setUnits(u=>u.map((uu,i)=>i===ui?{...uu,positions:(uu.positions||DEFAULT_PP_POS.map(p=>({...p}))).map((p,j)=>j===dragging.current?{x,y}:p)}:uu));
    };
    const onUp=()=>{dragging.current=null;window.removeEventListener("mousemove",onMove);window.removeEventListener("mouseup",onUp);window.removeEventListener("touchmove",onMove);window.removeEventListener("touchend",onUp);};
    window.addEventListener("mousemove",onMove);window.addEventListener("mouseup",onUp);
    window.addEventListener("touchmove",onMove,{passive:false});window.addEventListener("touchend",onUp);
  };
  const col=LINE_COLORS.find(c=>c.name===unit.color);
  const tokenBg=col?col.border:UDJ.maroon;
  return (
    <div style={{borderRadius:10,padding:"10px 12px",...sectionBg(unit.color),marginBottom:12}}>
      <LineHeader label={unit.label} setLabel={v=>setUnits(u=>u.map((uu,i)=>i===ui?{...uu,label:v}:uu))}
        color={unit.color} setColor={v=>setUnits(u=>u.map((uu,i)=>i===ui?{...uu,color:v}:uu))}
        notes={unit.notes||""} setNotes={v=>setUnits(u=>u.map((uu,i)=>i===ui?{...uu,notes:v}:uu))}
        canRemove={false}/>
      <p style={{fontSize:11,color:"#888",margin:"0 0 8px"}}>Drag skaters to set your formation</p>
      <div style={{display:"flex",gap:12,alignItems:"flex-start",flexWrap:"wrap"}}>
        <div ref={rinkRef} style={{position:"relative",width:OZ_W,height:OZ_H,borderRadius:10,overflow:"hidden",flexShrink:0,border:"1px solid #90c8de",userSelect:"none",touchAction:"none"}}>
          <OffensiveZone/>
          {positions.map((pos,si)=>{
            const p=unit.slots[si];
            return (
              <div key={si} onMouseDown={e=>startDrag(e,si)} onTouchStart={e=>startDrag(e,si)}
                style={{position:"absolute",left:`${pos.x}%`,top:`${pos.y}%`,transform:"translate(-50%,-50%)",zIndex:10,userSelect:"none",display:"flex",flexDirection:"column",alignItems:"center",gap:2,cursor:"grab"}}>
                <div style={{width:38,height:38,borderRadius:"50%",background:p?tokenBg:"rgba(255,255,255,0.92)",border:p?"2px solid rgba(0,0,0,0.15)":`2px dashed ${UDJ.maroon}`,display:"flex",alignItems:"center",justifyContent:"center",boxShadow:"0 2px 6px rgba(0,0,0,0.2)"}}>
                  {p?<span style={{fontSize:11,fontWeight:700,color:UDJ.white,textAlign:"center",lineHeight:1}}>#{p.number}</span>
                    :<span style={{fontSize:10,color:UDJ.maroon,fontWeight:700}}>{si+1}</span>}
                </div>
                {p&&<div style={{fontSize:9,fontWeight:600,color:UDJ.white,background:"rgba(0,0,0,0.65)",borderRadius:4,padding:"1px 5px",maxWidth:58,textAlign:"center",whiteSpace:"nowrap",overflow:"hidden",textOverflow:"ellipsis"}}>{p.name}</div>}
              </div>
            );
          })}
        </div>
        <div style={{display:"flex",flexDirection:"column",gap:6,minWidth:150,flex:1}}>
          {unit.slots.map((p,si)=>(
            <SlotEl key={si} label={`Skater ${si+1}`} player={p}
              setter={val=>setUnits(u=>u.map((uu,i)=>i===ui?{...uu,slots:uu.slots.map((s,j)=>j===si?val:s)}:uu))}/>
          ))}
        </div>
      </div>
    </div>
  );
}

// ── Print Summary ─────────────────────────────────────────────────────────────
function pName(p){return p?`#${p.number} ${p.name} (${p.pos}/${p.hand})`:"—";}

function PrintSummary({ esLines,esPairs,goalie1,goalie2,ppUnits,pkUnits,eaUnits,gameInfo,players,scratches }) {
  const scratchedPlayers = players.filter(p=>scratches[p.id]==="scratch");
  const injuredPlayers   = players.filter(p=>scratches[p.id]==="injured");

  const doPrint=()=>{
    const el=document.getElementById("print-area");
    const w=window.open("","_blank");
    w.document.write(`<html><head><title>Lineup – U of D Jesuit Hockey</title><style>
      *{box-sizing:border-box}
      body{font-family:Arial,sans-serif;padding:18px 24px;color:#2C0A0E;margin:0;font-size:12px}
      .hdr{background:#6B1F2A;color:#fff;padding:12px 18px;border-radius:6px;margin-bottom:12px;display:flex;justify-content:space-between;align-items:center}
      .hdr h1{margin:0;font-size:18px}.hdr .sub{font-size:11px;opacity:0.75;margin-top:2px}
      .gi{display:grid;grid-template-columns:repeat(3,1fr);gap:8px;margin-bottom:12px;padding:8px 12px;border:1px solid #C8A000;border-radius:5px;background:#FFFDF0}
      .gi-lbl{font-size:8px;text-transform:uppercase;font-weight:700;color:#8B6000;letter-spacing:.06em}.gi-val{font-size:12px;font-weight:700}
      .sec{font-size:11px;background:#6B1F2A;color:#fff;padding:3px 10px;border-radius:4px;margin:10px 0 5px;letter-spacing:.04em;text-transform:uppercase;font-weight:700}
      .cols{display:grid;gap:0 16px}
      .block{margin-bottom:7px;padding-left:8px;page-break-inside:avoid;border-left:2px solid #D8C8CA}
      .blabel{font-weight:700;font-size:11px;margin-bottom:1px;color:#6B1F2A}
      .brow{font-size:11px;line-height:1.75}
      .note{font-size:9px;color:#6B1F2A;font-style:italic;margin-top:1px}
      .scratch-row{display:flex;flex-wrap:wrap;gap:6px;padding-left:8px;margin-bottom:4px}
      .badge{font-size:10px;padding:2px 7px;border-radius:4px;font-weight:700}
      .scr{background:#6B1F2A;color:#fff}.inj{background:#B8860B;color:#fff}
      @media print{body{padding:10px 14px}@page{margin:0.4in}}
    </style></head><body>${el.innerHTML}</body></html>`);
    w.document.close();w.focus();w.print();
  };

  const UnitCol = ({unit}) => (
    <div className="block" style={{marginBottom:7,paddingLeft:8,borderLeft:`2px solid #D8C8CA`,pageBreakInside:"avoid"}}>
      <div style={{fontWeight:700,fontSize:11,marginBottom:1,color:"#6B1F2A"}}>{unit.label}</div>
      {unit.slots.map((p,i)=><div key={i} style={{fontSize:11,lineHeight:1.75}}><b>S{i+1}:</b> {pName(p)}</div>)}
      {unit.notes&&<div style={{fontSize:9,color:"#6B1F2A",fontStyle:"italic",marginTop:1}}>{unit.notes}</div>}
    </div>
  );

  return (
    <div>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:16}}>
        <div>
          <h2 style={{margin:0,fontSize:16,fontWeight:700,color:UDJ.maroon}}>Print Summary</h2>
          <p style={{margin:0,fontSize:12,color:"#888"}}>Review your full lineup, then print or save as PDF</p>
        </div>
        <button onClick={doPrint} style={{padding:"8px 18px",borderRadius:7,cursor:"pointer",background:UDJ.maroon,color:UDJ.white,border:"none",fontSize:13,fontWeight:700,boxShadow:"0 2px 8px rgba(107,31,42,0.3)"}}>🖨 Print</button>
      </div>

      <div id="print-area" style={{fontFamily:"Arial,sans-serif",color:UDJ.text,fontSize:12}}>
        {/* Header */}
        <div style={{background:UDJ.maroon,color:UDJ.white,padding:"12px 18px",borderRadius:6,marginBottom:12,display:"flex",justifyContent:"space-between",alignItems:"center"}}>
          <div>
            <div style={{fontSize:18,fontWeight:700}}>University of Detroit Jesuit</div>
            <div style={{fontSize:12,opacity:0.8,marginTop:2}}>Hockey — Game Lineup</div>
          </div>
          <div style={{textAlign:"right",fontSize:11,opacity:0.75}}>Cubs Hockey · Detroit, MI</div>
        </div>

        {/* Game Info */}
        {(gameInfo.opponent||gameInfo.date||gameInfo.rink)&&(
          <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:8,marginBottom:12,padding:"8px 12px",border:"1px solid #C8A000",borderRadius:5,background:"#FFFDF0"}}>
            {[["Opponent",gameInfo.opponent],["Date",gameInfo.date],["Rink",gameInfo.rink]].map(([lbl,val])=>val?(
              <div key={lbl}><div style={{fontSize:8,textTransform:"uppercase",fontWeight:700,color:"#8B6000",letterSpacing:"0.06em"}}>{lbl}</div><div style={{fontSize:12,fontWeight:700}}>{val}</div></div>
            ):null)}
          </div>
        )}

        {/* Scratches / Injuries */}
        {(scratchedPlayers.length>0||injuredPlayers.length>0)&&(
          <>
            <div style={{fontSize:11,background:UDJ.maroon,color:"#fff",padding:"3px 10px",borderRadius:4,margin:"10px 0 6px",letterSpacing:"0.04em",textTransform:"uppercase",fontWeight:700}}>Scratches & Injuries</div>
            <div style={{display:"flex",flexWrap:"wrap",gap:6,paddingLeft:8,marginBottom:8}}>
              {scratchedPlayers.map(p=>(
                <span key={p.id} style={{fontSize:10,padding:"2px 8px",borderRadius:4,fontWeight:700,background:UDJ.maroon,color:"#fff"}}>SCR: #{p.number} {p.name}</span>
              ))}
              {injuredPlayers.map(p=>(
                <span key={p.id} style={{fontSize:10,padding:"2px 8px",borderRadius:4,fontWeight:700,background:"#B8860B",color:"#fff"}}>INJ: #{p.number} {p.name}</span>
              ))}
            </div>
          </>
        )}

        {/* Goalies */}
        <div style={{fontSize:11,background:UDJ.maroon,color:"#fff",padding:"3px 10px",borderRadius:4,margin:"10px 0 5px",letterSpacing:"0.04em",textTransform:"uppercase",fontWeight:700}}>Goalies</div>
        <div style={{paddingLeft:8,marginBottom:8,fontSize:11,lineHeight:1.9}}>
          <div><b>Starter:</b> {pName(goalie1)}</div>
          <div><b>Backup:</b> {pName(goalie2)}</div>
        </div>

        {/* Even Strength */}
        <div style={{fontSize:11,background:UDJ.maroon,color:"#fff",padding:"3px 10px",borderRadius:4,margin:"10px 0 5px",letterSpacing:"0.04em",textTransform:"uppercase",fontWeight:700}}>Even Strength</div>
        {esLines.map((line,i)=>(
          <div key={i} style={{marginBottom:7,paddingLeft:8,borderLeft:`2px solid ${UDJ.border}`,pageBreakInside:"avoid"}}>
            <div style={{fontWeight:700,fontSize:11,marginBottom:1,color:UDJ.maroon}}>{line.label}</div>
            <div style={{fontSize:11,lineHeight:1.75}}>LW: {pName(line.lw)} &nbsp;|&nbsp; C: {pName(line.c)} &nbsp;|&nbsp; RW: {pName(line.rw)}</div>
            {line.notes&&<div style={{fontSize:9,color:UDJ.maroon,fontStyle:"italic"}}>{line.notes}</div>}
          </div>
        ))}
        {esPairs.map((pair,i)=>(
          <div key={i} style={{marginBottom:7,paddingLeft:8,borderLeft:`2px solid ${UDJ.border}`,pageBreakInside:"avoid"}}>
            <div style={{fontWeight:700,fontSize:11,marginBottom:1,color:UDJ.maroon}}>{pair.label}</div>
            <div style={{fontSize:11,lineHeight:1.75}}>LD: {pName(pair.ld)} &nbsp;|&nbsp; RD: {pName(pair.rd)}</div>
            {pair.notes&&<div style={{fontSize:9,color:UDJ.maroon,fontStyle:"italic"}}>{pair.notes}</div>}
          </div>
        ))}

        {/* Power Play — 2 columns */}
        <div style={{fontSize:11,background:UDJ.maroon,color:"#fff",padding:"3px 10px",borderRadius:4,margin:"10px 0 5px",letterSpacing:"0.04em",textTransform:"uppercase",fontWeight:700}}>Power Play</div>
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:"0 16px"}}>
          {ppUnits.map((unit,i)=><UnitCol key={i} unit={unit}/>)}
        </div>

        {/* Penalty Kill — 3 columns */}
        <div style={{fontSize:11,background:UDJ.maroon,color:"#fff",padding:"3px 10px",borderRadius:4,margin:"10px 0 5px",letterSpacing:"0.04em",textTransform:"uppercase",fontWeight:700}}>Penalty Kill</div>
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:"0 16px"}}>
          {pkUnits.map((unit,i)=><UnitCol key={i} unit={unit}/>)}
        </div>

        {/* Extra Attacker — 2 columns */}
        <div style={{fontSize:11,background:UDJ.maroon,color:"#fff",padding:"3px 10px",borderRadius:4,margin:"10px 0 5px",letterSpacing:"0.04em",textTransform:"uppercase",fontWeight:700}}>Extra Attacker</div>
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:"0 16px"}}>
          {eaUnits.map((unit,i)=><UnitCol key={i} unit={unit}/>)}
        </div>

        <div style={{marginTop:14,paddingTop:8,borderTop:`1px solid ${UDJ.border}`,fontSize:9,color:"#aaa",textAlign:"center"}}>University of Detroit Jesuit Hockey · Lineup Manager</div>
      </div>
    </div>
  );
}

// ── Main App ──────────────────────────────────────────────────────────────────
const INIT = () => ({
  players:[
    {id:1,name:"Smith",  number:"7", pos:"LW",hand:"L"},
    {id:2,name:"Jones",  number:"12",pos:"C", hand:"R"},
    {id:3,name:"Miller", number:"19",pos:"RW",hand:"R"},
    {id:4,name:"Davis",  number:"4", pos:"LD",hand:"L"},
    {id:5,name:"Wilson", number:"22",pos:"RD",hand:"R"},
    {id:6,name:"Taylor", number:"30",pos:"G", hand:"L"},
    {id:7,name:"Anderson",number:"8",pos:"LW",hand:"L"},
    {id:8,name:"Thomas", number:"11",pos:"C", hand:"R"},
    {id:9,name:"Jackson",number:"17",pos:"RW",hand:"L"},
    {id:10,name:"White", number:"3", pos:"LD",hand:"R"},
    {id:11,name:"Harris",number:"25",pos:"RD",hand:"L"},
    {id:12,name:"Cooper",number:"31",pos:"G", hand:"L"},
  ],
  esLines:[mkLine("Line 1"),mkLine("Line 2")],
  esPairs:[mkPair("Pair 1"),mkPair("Pair 2")],
  goalie1:null,goalie2:null,
  ppUnits:[mkUnit(5,"PP1"),mkUnit(5,"PP2")],
  pkUnits:[mkUnit(4,"PK1"),mkUnit(4,"PK2"),mkUnit(4,"PK3")],
  eaUnits:[mkUnit(6,"EA1"),mkUnit(6,"EA2")],
  gameInfo:{opponent:"",date:"",rink:""},
  scratches:{},
});

export default function App() {
  const [state,setState]=useState(INIT());
  const {players,esLines,esPairs,goalie1,goalie2,ppUnits,pkUnits,eaUnits,gameInfo,scratches}=state;
  const set=(key)=>(val)=>setState(s=>({...s,[key]:typeof val==="function"?val(s[key]):val}));
  const setPlayers=set("players"),setEsLines=set("esLines"),setEsPairs=set("esPairs");
  const setGoalie1=set("goalie1"),setGoalie2=set("goalie2");
  const setPpUnits=set("ppUnits"),setPkUnits=set("pkUnits"),setEaUnits=set("eaUnits");
  const setGameInfo=(k,v)=>setState(s=>({...s,gameInfo:{...s.gameInfo,[k]:v}}));
  const setScratches=set("scratches");

  const toggleScratch=(id,type)=>{
    setScratches(sc=>({...sc,[id]:sc[id]===type?undefined:type}));
  };

  const [form,setForm]=useState({name:"",number:"",pos:"LW",hand:"L"});
  const [tab,setTab]=useState(0);
  const [selected,setSelected]=useState(null);
  const [savedLineups,setSavedLineups]=useState([]);
  const [saveName,setSaveName]=useState("");
  const [showSavePanel,setShowSavePanel]=useState(false);
  const [saveStatus,setSaveStatus]=useState("");

  useEffect(()=>{
    (async()=>{
      try {
        const r=await window.storage.list("lineup:");
        const items=await Promise.all((r?.keys||[]).map(async k=>{
          const d=await window.storage.get(k);
          return d?{key:k,name:k.replace("lineup:",""),data:JSON.parse(d.value)}:null;
        }));
        setSavedLineups(items.filter(Boolean).sort((a,b)=>a.name.localeCompare(b.name)));
      } catch(e){}
    })();
  },[]);

  const saveLineup=async()=>{
    if(!saveName.trim())return;
    const key=`lineup:${saveName.trim()}`;
    try {
      await window.storage.set(key,JSON.stringify(state));
      const entry={key,name:saveName.trim(),data:state};
      setSavedLineups(prev=>[...prev.filter(x=>x.key!==key),entry].sort((a,b)=>a.name.localeCompare(b.name)));
      setSaveName("");setSaveStatus("✓ Saved!");setTimeout(()=>setSaveStatus(""),2500);
    } catch(e){setSaveStatus("Save failed");}
  };
  const loadLineup=(item)=>{setState(item.data);setSaveStatus(`✓ Loaded: ${item.name}`);setTimeout(()=>setSaveStatus(""),2500);setShowSavePanel(false);};
  const deleteLineup=async(item,e)=>{e.stopPropagation();try{await window.storage.delete(item.key);setSavedLineups(prev=>prev.filter(x=>x.key!==item.key));}catch(e){}};

  const addPlayer=()=>{
    if(!form.name.trim()||!form.number.trim())return;
    setPlayers(p=>[...p,{id:Date.now(),...form}]);
    setForm(f=>({...f,name:"",number:""}));
  };

  const clearFromAll=(id)=>{
    setEsLines(ls=>ls.map(l=>({...l,lw:l.lw?.id===id?null:l.lw,c:l.c?.id===id?null:l.c,rw:l.rw?.id===id?null:l.rw})));
    setEsPairs(ps=>ps.map(p=>({...p,ld:p.ld?.id===id?null:p.ld,rd:p.rd?.id===id?null:p.rd})));
    if(goalie1?.id===id)setGoalie1(null);if(goalie2?.id===id)setGoalie2(null);
    setPpUnits(u=>u.map(uu=>({...uu,slots:uu.slots.map(s=>s?.id===id?null:s)})));
    setPkUnits(u=>u.map(uu=>({...uu,slots:uu.slots.map(s=>s?.id===id?null:s)})));
    setEaUnits(u=>u.map(uu=>({...uu,slots:uu.slots.map(s=>s?.id===id?null:s)})));
  };
  const removePlayer=(id)=>{setPlayers(p=>p.filter(x=>x.id!==id));clearFromAll(id);};

  const assignedIds=()=>{
    const ids=new Set();
    esLines.forEach(l=>{l.lw&&ids.add(l.lw.id);l.c&&ids.add(l.c.id);l.rw&&ids.add(l.rw.id);});
    esPairs.forEach(p=>{p.ld&&ids.add(p.ld.id);p.rd&&ids.add(p.rd.id);});
    if(goalie1)ids.add(goalie1.id);if(goalie2)ids.add(goalie2.id);
    [ppUnits,pkUnits,eaUnits].forEach(us=>us.forEach(u=>u.slots.forEach(s=>s&&ids.add(s.id))));
    return ids;
  };

  const handleRosterClick=(player)=>{
    if(selected?.type==="slot"){selected.data.setter(player);setSelected(null);}
    else setSelected({type:"roster",data:player});
  };
  const makeSlotClick=(player,setter)=>()=>{
    if(selected?.type==="roster"){setter(selected.data);setSelected(null);}
    else if(selected?.type==="slot"){
      if(selected.data.setter===setter){setSelected(null);return;}
      setter(selected.data.player);selected.data.setter(player);setSelected(null);
    } else setSelected({type:"slot",data:{player,setter}});
  };
  const makeSlotDrop=(setter)=>(e)=>{
    const p=players.find(x=>x.id===parseInt(e.dataTransfer.getData("playerId")));
    if(p){setter(p);setSelected(null);}
  };
  const makeDrag=(player)=>(e)=>e.dataTransfer.setData("playerId",player.id);

  const SlotEl=({label,player,setter})=>(
    <Slot label={label} player={player}
      selected={selected?.type==="slot"&&selected.data.setter===setter}
      onClick={makeSlotClick(player,setter)} onDrop={makeSlotDrop(setter)}/>
  );

  const sectionBg=(color)=>{
    const c=LINE_COLORS.find(x=>x.name===color);
    return c?{background:c.bg,border:`1px solid ${c.border}`}:{background:UDJ.lgray,border:`1px solid ${UDJ.border}`};
  };

  const renderUnitGrid=(units,setUnits,size)=>(
    <div style={{display:"flex",flexDirection:"column",gap:12}}>
      {units.map((unit,ui)=>(
        <div key={ui} style={{borderRadius:10,padding:"10px 12px",...sectionBg(unit.color)}}>
          <LineHeader label={unit.label} setLabel={v=>setUnits(u=>u.map((uu,i)=>i===ui?{...uu,label:v}:uu))}
            color={unit.color} setColor={v=>setUnits(u=>u.map((uu,i)=>i===ui?{...uu,color:v}:uu))}
            notes={unit.notes||""} setNotes={v=>setUnits(u=>u.map((uu,i)=>i===ui?{...uu,notes:v}:uu))}
            canRemove={false}/>
          <div style={{display:"grid",gridTemplateColumns:`repeat(${Math.min(size,3)},1fr)`,gap:8}}>
            {unit.slots.map((p,si)=>(
              <SlotEl key={si} label={`Skater ${si+1}`} player={p}
                setter={val=>setUnits(u=>u.map((uu,i)=>i===ui?{...uu,slots:uu.slots.map((s,j)=>j===si?val:s)}:uu))}/>
            ))}
          </div>
        </div>
      ))}
    </div>
  );

  const assigned=assignedIds();

  return (
    <div style={{fontFamily:"Arial,sans-serif",maxWidth:760,background:UDJ.white,minHeight:"100vh"}}>
      {/* Header */}
      <div style={{background:UDJ.maroon,padding:"14px 20px",marginBottom:14,borderRadius:"0 0 10px 10px",display:"flex",justifyContent:"space-between",alignItems:"center"}}>
        <div>
          <div style={{color:UDJ.white,fontWeight:700,fontSize:17,letterSpacing:"0.02em"}}>University of Detroit Jesuit</div>
          <div style={{color:"rgba(255,255,255,0.75)",fontWeight:500,fontSize:13,marginTop:2}}>Hockey Lineup Manager</div>
        </div>
        <div style={{display:"flex",gap:8,alignItems:"center"}}>
          {saveStatus&&<span style={{fontSize:12,color:"rgba(255,255,255,0.85)",fontWeight:600}}>{saveStatus}</span>}
          <button onClick={()=>setShowSavePanel(s=>!s)} style={{padding:"6px 14px",borderRadius:6,cursor:"pointer",background:"rgba(255,255,255,0.18)",color:UDJ.white,border:"1px solid rgba(255,255,255,0.3)",fontSize:12,fontWeight:600}}>
            💾 Lineups
          </button>
        </div>
      </div>

      <div style={{padding:"0 4px"}}>
        {/* Save/Load Panel */}
        {showSavePanel&&(
          <div style={{marginBottom:12,padding:"12px 14px",background:UDJ.maroonLight,border:`1px solid ${UDJ.border}`,borderRadius:10}}>
            <div style={{fontSize:13,fontWeight:700,color:UDJ.maroon,marginBottom:8}}>Save / Load Lineups</div>
            <div style={{display:"flex",gap:8,marginBottom:10}}>
              <input value={saveName} onChange={e=>setSaveName(e.target.value)} placeholder='Name (e.g. "vs Cranbrook 4/18")'
                onKeyDown={e=>e.key==="Enter"&&saveLineup()}
                style={{flex:1,padding:"5px 10px",fontSize:12,borderRadius:6,border:`1px solid ${UDJ.border}`}}/>
              <button onClick={saveLineup} style={{padding:"5px 14px",borderRadius:6,cursor:"pointer",background:UDJ.maroon,color:UDJ.white,border:"none",fontSize:12,fontWeight:700}}>Save</button>
            </div>
            {savedLineups.length===0
              ? <div style={{fontSize:12,color:"#aaa"}}>No saved lineups yet.</div>
              : <div style={{display:"flex",flexDirection:"column",gap:5,maxHeight:180,overflowY:"auto"}}>
                  {savedLineups.map(item=>(
                    <div key={item.key} onClick={()=>loadLineup(item)}
                      style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"6px 10px",borderRadius:7,background:UDJ.white,border:`1px solid ${UDJ.border}`,cursor:"pointer"}}>
                      <span style={{fontSize:12,fontWeight:600,color:UDJ.maroon}}>📋 {item.name}</span>
                      <div style={{display:"flex",gap:6,alignItems:"center"}}>
                        <span style={{fontSize:11,color:"#aaa"}}>tap to load</span>
                        <button onClick={e=>deleteLineup(item,e)} style={{fontSize:10,padding:"1px 7px",borderRadius:4,cursor:"pointer",border:`1px solid ${UDJ.border}`,color:UDJ.maroon,background:"transparent"}}>✕</button>
                      </div>
                    </div>
                  ))}
                </div>
            }
          </div>
        )}

        {/* Game Info */}
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:8,marginBottom:12,padding:"10px 14px",background:"#FFFDF0",border:"1px solid #C8A000",borderRadius:10}}>
          {[["Opponent","opponent","vs. Team Name"],["Date","date","e.g. April 18, 2026"],["Rink","rink","e.g. Cass Tech Arena"]].map(([lbl,key,ph])=>(
            <div key={key}>
              <div style={{fontSize:9,fontWeight:700,textTransform:"uppercase",letterSpacing:"0.07em",color:"#8B6000",marginBottom:3}}>{lbl}</div>
              <input value={gameInfo[key]} onChange={e=>setGameInfo(key,e.target.value)} placeholder={ph}
                style={{width:"100%",padding:"4px 8px",fontSize:12,borderRadius:5,border:`1px solid ${UDJ.border}`,fontWeight:600,color:UDJ.text,boxSizing:"border-box"}}/>
            </div>
          ))}
        </div>

        <div style={{display:"grid",gridTemplateColumns:tab===4?"1fr":"1fr 220px",gap:16,alignItems:"start"}}>
          <div>
            <div style={{display:"flex",gap:5,marginBottom:14,flexWrap:"wrap"}}>
              {TABS.map((t,i)=>(
                <button key={i} onClick={()=>setTab(i)} style={{padding:"6px 13px",borderRadius:6,fontSize:12,cursor:"pointer",fontWeight:700,
                  background:tab===i?UDJ.maroon:UDJ.white,
                  color:tab===i?UDJ.white:UDJ.maroon,
                  border:tab===i?"none":`1px solid ${UDJ.border}`}}>
                  {i===4?"🖨 "+t:t}
                </button>
              ))}
            </div>

            {tab===0&&(
              <div style={{display:"flex",flexDirection:"column",gap:12}}>
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center"}}>
                  <span style={{fontSize:13,fontWeight:700,color:UDJ.maroon}}>Forward Lines</span>
                  <button onClick={()=>setEsLines(l=>[...l,mkLine(`Line ${l.length+1}`)])} style={{fontSize:12,padding:"3px 12px",borderRadius:6,cursor:"pointer",border:`1px solid ${UDJ.border}`,background:UDJ.white,fontWeight:600,color:UDJ.maroon}}>+ Add line</button>
                </div>
                {esLines.map((line,li)=>(
                  <div key={li} style={{borderRadius:10,padding:"10px 12px",...sectionBg(line.color)}}>
                    <LineHeader label={line.label} setLabel={v=>setEsLines(ls=>ls.map((l,i)=>i===li?{...l,label:v}:l))}
                      color={line.color} setColor={v=>setEsLines(ls=>ls.map((l,i)=>i===li?{...l,color:v}:l))}
                      notes={line.notes||""} setNotes={v=>setEsLines(ls=>ls.map((l,i)=>i===li?{...l,notes:v}:l))}
                      onRemove={()=>setEsLines(ls=>ls.filter((_,i)=>i!==li))} canRemove={esLines.length>1}/>
                    <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:8}}>
                      <SlotEl label="LW" player={line.lw} setter={v=>setEsLines(ls=>ls.map((l,i)=>i===li?{...l,lw:v}:l))}/>
                      <SlotEl label="C"  player={line.c}  setter={v=>setEsLines(ls=>ls.map((l,i)=>i===li?{...l,c:v}:l))}/>
                      <SlotEl label="RW" player={line.rw} setter={v=>setEsLines(ls=>ls.map((l,i)=>i===li?{...l,rw:v}:l))}/>
                    </div>
                  </div>
                ))}
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginTop:4}}>
                  <span style={{fontSize:13,fontWeight:700,color:UDJ.maroon}}>Defensive Pairings</span>
                  <button onClick={()=>setEsPairs(p=>[...p,mkPair(`Pair ${p.length+1}`)])} style={{fontSize:12,padding:"3px 12px",borderRadius:6,cursor:"pointer",border:`1px solid ${UDJ.border}`,background:UDJ.white,fontWeight:600,color:UDJ.maroon}}>+ Add pair</button>
                </div>
                {esPairs.map((pair,pi)=>(
                  <div key={pi} style={{borderRadius:10,padding:"10px 12px",...sectionBg(pair.color)}}>
                    <LineHeader label={pair.label} setLabel={v=>setEsPairs(ps=>ps.map((p,i)=>i===pi?{...p,label:v}:p))}
                      color={pair.color} setColor={v=>setEsPairs(ps=>ps.map((p,i)=>i===pi?{...p,color:v}:p))}
                      notes={pair.notes||""} setNotes={v=>setEsPairs(ps=>ps.map((p,i)=>i===pi?{...p,notes:v}:p))}
                      onRemove={()=>setEsPairs(ps=>ps.filter((_,i)=>i!==pi))} canRemove={esPairs.length>1}/>
                    <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:8}}>
                      <SlotEl label="LD" player={pair.ld} setter={v=>setEsPairs(ps=>ps.map((p,i)=>i===pi?{...p,ld:v}:p))}/>
                      <SlotEl label="RD" player={pair.rd} setter={v=>setEsPairs(ps=>ps.map((p,i)=>i===pi?{...p,rd:v}:p))}/>
                    </div>
                  </div>
                ))}
                <div style={{background:UDJ.lgray,borderRadius:10,padding:"10px 12px",border:`1px solid ${UDJ.border}`}}>
                  <div style={{fontSize:13,fontWeight:700,color:UDJ.maroon,marginBottom:8}}>Goalies</div>
                  <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:8}}>
                    <SlotEl label="Starter" player={goalie1} setter={setGoalie1}/>
                    <SlotEl label="Backup"  player={goalie2} setter={setGoalie2}/>
                  </div>
                </div>
              </div>
            )}
            {tab===1&&<div>{ppUnits.map((unit,ui)=><PPUnit key={ui} unit={unit} ui={ui} setUnits={setPpUnits} sectionBg={sectionBg} SlotEl={SlotEl}/>)}</div>}
            {tab===2&&renderUnitGrid(pkUnits,setPkUnits,4)}
            {tab===3&&renderUnitGrid(eaUnits,setEaUnits,6)}
            {tab===4&&<PrintSummary esLines={esLines} esPairs={esPairs} goalie1={goalie1} goalie2={goalie2} ppUnits={ppUnits} pkUnits={pkUnits} eaUnits={eaUnits} gameInfo={gameInfo} players={players} scratches={scratches}/>}
          </div>

          {tab!==4&&(
            <div style={{position:"sticky",top:8}}>
              <div style={{fontSize:13,fontWeight:700,color:UDJ.maroon,marginBottom:8}}>Roster</div>
              <div style={{background:UDJ.lgray,borderRadius:10,padding:"10px",border:`1px solid ${UDJ.border}`,marginBottom:10}}>
                <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:6,marginBottom:8}}>
                  <input placeholder="Name" value={form.name} onChange={e=>setForm(f=>({...f,name:e.target.value}))}
                    style={{padding:"4px 8px",fontSize:12,borderRadius:6,border:`1px solid ${UDJ.border}`,gridColumn:"1/-1"}}/>
                  <input placeholder="#" value={form.number} onChange={e=>setForm(f=>({...f,number:e.target.value}))}
                    style={{padding:"4px 8px",fontSize:12,borderRadius:6,border:`1px solid ${UDJ.border}`}}/>
                  <select value={form.pos} onChange={e=>setForm(f=>({...f,pos:e.target.value}))}
                    style={{padding:"4px 6px",fontSize:12,borderRadius:6,border:`1px solid ${UDJ.border}`}}>
                    {POSITIONS.map(p=><option key={p}>{p}</option>)}
                  </select>
                  <select value={form.hand} onChange={e=>setForm(f=>({...f,hand:e.target.value}))}
                    style={{padding:"4px 6px",fontSize:12,borderRadius:6,border:`1px solid ${UDJ.border}`}}>
                    {HANDS.map(h=><option key={h} value={h}>{h==="L"?"L (left)":"R (right)"}</option>)}
                  </select>
                </div>
                <button onClick={addPlayer} style={{width:"100%",padding:"5px 0",fontSize:12,borderRadius:6,cursor:"pointer",border:"none",background:UDJ.maroon,color:UDJ.white,fontWeight:700}}>+ Add Player</button>
              </div>

              {/* Scratch/Injury legend */}
              <div style={{display:"flex",gap:6,marginBottom:6,fontSize:10,color:"#888"}}>
                <span>Long-press to mark:</span>
                <span style={{padding:"1px 6px",borderRadius:3,background:UDJ.maroon,color:"#fff",fontWeight:700}}>SCR</span>
                <span style={{padding:"1px 6px",borderRadius:3,background:"#B8860B",color:"#fff",fontWeight:700}}>INJ</span>
              </div>

              <div style={{display:"flex",flexDirection:"column",gap:5,maxHeight:400,overflowY:"auto"}}>
                {players.map(p=>{
                  const st=scratches[p.id];
                  return (
                    <div key={p.id} style={{position:"relative"}}>
                      <RosterCard player={p} assigned={assigned.has(p.id)}
                        selected={selected?.type==="roster"&&selected.data.id===p.id}
                        scratchStatus={st}
                        onDrag={makeDrag(p)} onClick={()=>handleRosterClick(p)}/>
                      <div style={{position:"absolute",top:4,right:4,display:"flex",gap:3}}>
                        <button onClick={e=>{e.stopPropagation();toggleScratch(p.id,"scratch");}}
                          style={{fontSize:9,padding:"1px 5px",borderRadius:3,cursor:"pointer",border:"none",background:st==="scratch"?UDJ.maroon:"#eee",color:st==="scratch"?"#fff":"#888",fontWeight:700}}>S</button>
                        <button onClick={e=>{e.stopPropagation();toggleScratch(p.id,"injured");}}
                          style={{fontSize:9,padding:"1px 5px",borderRadius:3,cursor:"pointer",border:"none",background:st==="injured"?"#B8860B":"#eee",color:st==="injured"?"#fff":"#888",fontWeight:700}}>I</button>
                        <button onClick={e=>{e.stopPropagation();removePlayer(p.id);}}
                          style={{fontSize:9,padding:"1px 5px",borderRadius:3,cursor:"pointer",border:"none",background:"transparent",color:"#ccc",fontWeight:700}}>✕</button>
                      </div>
                    </div>
                  );
                })}
              </div>
              {selected&&(
                <div style={{marginTop:10,padding:"7px 10px",borderRadius:6,background:UDJ.maroonLight,border:`1px solid ${UDJ.maroon}`,fontSize:12,color:UDJ.maroon,fontWeight:600}}>
                  {selected.type==="roster"?`${selected.data.name} selected — tap a slot`:`Swapping ${selected.data.player?.name||"empty"} — tap another slot`}
                  <button onClick={()=>setSelected(null)} style={{marginLeft:8,fontSize:11,cursor:"pointer",border:"none",background:"transparent",color:UDJ.maroon,fontWeight:700}}>cancel</button>
                </div>
              )}
            </div>
          )}
        </div>
      </div>
    </div>
  );
}
