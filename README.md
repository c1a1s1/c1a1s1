import React, {
  useState, useEffect, useRef, useCallback, useMemo, useReducer, memo
} from 'react';
import {
  Flame, ListOrdered, VolumeX, Volume2, Lock, Skull,
  Trophy, Award, BookOpen, ChevronLeft, Trash2, Brain,
  Glasses, BarChartBig, LogOut, Siren, FastForward, Scan,
  Hexagon, Eye, Gavel, Crosshair, Star, AlertTriangle, PackageOpen, Dumbbell, Zap as SynergyIcon, Ghost, ShieldAlert,
  Map as MapIcon, Library, FlaskConical, Swords, Info
} from 'lucide-react';

// ── AUDIO CONFIG ─────────────────────────────────────────────────────────────
const AUDIO_ASSETS = {
  hit: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/attack_game_sound.mp3',
  lose: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/vocal%20sad%20own.mp3',
  
  bgm_menu: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/Menu%20music.mp3',
  bgm_explore: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/game%20music%20loop1.mp3',
  bgm_battle: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/game%20music%20loop2.mp3',
  bgm_dark: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/Dangerous%20music%20for%20dark%20parts.mp3',
  
  heartbeat: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/heart_beat.mp3',
  crowd_riot: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/vocal%20crowd_riot%20loop.mp3',
  sirens: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/police_sirens.mp3',
  welcome: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/vocal%20Welcome_to_the_underground.mp3',
  new_game: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/vocal%20New_game.mp3',
  good_luck: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/vocal%20Good_luck_kid.mp3',
  boss: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/vocal%20%20boss!!!.mp3',
  all_in: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/vocal%20All_in_.mp3',
  illegal: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/vocal%20Hey_thats_illegal.mp3',
  vocal_win: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/vocal%20Haha,_what_a_win.mp3',
  game_over: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/vocal%20game_over.mp3',
  loser: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/vocal%20loser!!!.mp3',
  losing: 'https://ufmilirifpdrupzvopak.supabase.co/storage/v1/object/public/sounds/vocal%20losing.mp3'
};

// ── CONSTANTS (frozen) ───────────────────────────────────────────────────────
const MOVES = Object.freeze({ R: '✊', P: '✋', S: '✌️', N: '🌟' });
const DNA_COLORS = Object.freeze({ R: 'bg-stone-600', P: 'bg-emerald-600', S: 'bg-orange-600', '?': 'bg-fuchsia-600', N: 'bg-indigo-600' });
const MUTANT_EMOJIS = Object.freeze(['👺','👽','🤖','👾','👻','🧟','🧛','🧙','🐉','🦖']);

const INITIAL_CARDS = Object.freeze([
  { id:'c1',  name:'Rocky Rhino',     emoji:'🦏', rock:80,  paper:10, scissors:10, pwr:20, iq:10,  grit:50, flavor:'Brute force incarnate.',                   bg:'bg-gradient-to-br from-stone-500 to-stone-800',   rarity:'Common' },
  { id:'c5',  name:'Granny Granite',  emoji:'👵', rock:100, paper:0,  scissors:0,  pwr:35, iq:30,  grit:60, flavor:'She baked you cookies made of concrete.',    bg:'bg-gradient-to-br from-slate-500 to-slate-800',   rarity:'Rare',   passiveId:'SCISSOR_IMMUNE', passiveDesc:'Immune to Scissors.' },
  { id:'c8',  name:'Kid Boulder',     emoji:'👦', rock:65,  paper:15, scissors:20, pwr:15, iq:15,  grit:30, flavor:'Found a cool rock.',                        bg:'bg-gradient-to-br from-zinc-500 to-zinc-800',    rarity:'Common' },
  { id:'c2',  name:'Paper Panda',     emoji:'🐼', rock:10,  paper:80, scissors:10, pwr:15, iq:40,  grit:10, flavor:'Will fight you after this nap.',            bg:'bg-gradient-to-br from-emerald-400 to-emerald-700',rarity:'Common' },
  { id:'c6',  name:'Origami Owl',     emoji:'🦉', rock:5,   paper:90, scissors:5,  pwr:25, iq:70,  grit:15, flavor:'Silent, deadly, and highly flammable.',     bg:'bg-gradient-to-br from-teal-400 to-teal-700',    rarity:'Rare',   passiveId:'WIN_TIES',       passiveDesc:'Auto-wins Ties.' },
  { id:'c9',  name:'Magic Book',      emoji:'📘', rock:15,  paper:70, scissors:15, pwr:10, iq:80,  grit:20, flavor:'Contains forbidden knowledge of papercuts.',bg:'bg-gradient-to-br from-cyan-400 to-cyan-700',    rarity:'Common' },
  { id:'c3',  name:'Snippy Squirrel', emoji:'🐿️', rock:15,  paper:15, scissors:70, pwr:25, iq:45,  grit:40, flavor:'Chomps like nuts!',                        bg:'bg-gradient-to-br from-amber-500 to-orange-700', rarity:'Common' },
  { id:'c7',  name:'Scissor Snake',   emoji:'🐍', rock:5,   paper:5,  scissors:90, pwr:30, iq:60,  grit:70, flavor:'Snips your hopes.',                        bg:'bg-gradient-to-br from-lime-500 to-green-800',   rarity:'Rare' },
  { id:'c10', name:'Crab Claw',       emoji:'🦀', rock:20,  paper:10, scissors:70, pwr:20, iq:20,  grit:40, flavor:'Apex crustacean engineering.',             bg:'bg-gradient-to-br from-red-500 to-rose-800',     rarity:'Common' },
  { id:'c4',  name:'Raccoon',         emoji:'🦝', rock:33,  paper:33, scissors:34, pwr:15, iq:50,  grit:60, flavor:'Gambles everything.',                      bg:'bg-gradient-to-br from-fuchsia-500 to-purple-800',rarity:'Epic',   passiveId:'SECOND_CHANCE',  passiveDesc:'50% chance to reroll loss.' },
  { id:'c11', name:'Scanner Bot',     emoji:'🤖', rock:33,  paper:34, scissors:33, pwr:15, iq:95,  grit:10, flavor:'Processes permutations faster than you.',  bg:'bg-gradient-to-br from-blue-500 to-indigo-800',  rarity:'Epic',   passiveId:'SCOUTER',        passiveDesc:'Reveals hidden stats.' },
  { id:'c12', name:'Golem Guard',     emoji:'🗿', rock:85,  paper:5,  scissors:10, pwr:30, iq:20,  grit:80, flavor:'Ancient guardian.',                        bg:'bg-gradient-to-br from-stone-600 to-stone-900',  rarity:'Rare',   passiveId:'BLOCK_S',        passiveDesc:'Enemy cannot throw Scissors.' },
  { id:'c13', name:'Meteorite',       emoji:'☄️', rock:100, paper:0,  scissors:0,  pwr:40, iq:5,   grit:90, flavor:'From space, straight to your face.',       bg:'bg-gradient-to-br from-orange-700 to-stone-900', rarity:'Epic',   passiveId:'WIN_TIES',       passiveDesc:'Auto-wins Ties.' },
  { id:'c14', name:'Mummy Wrapper',   emoji:'🧻', rock:10,  paper:85, scissors:5,  pwr:15, iq:30,  grit:40, flavor:'Wraps you in bureaucracy.',                bg:'bg-gradient-to-br from-yellow-100 to-yellow-600',rarity:'Common' },
  { id:'c15', name:'Wind Spirit',     emoji:'🌪️', rock:0,   paper:95, scissors:5,  pwr:20, iq:85,  grit:20, flavor:'Ultimate counter to heavy rock.',          bg:'bg-gradient-to-br from-teal-200 to-cyan-500',    rarity:'Rare' },
  { id:'c16', name:'Origami Dragon',  emoji:'🐉', rock:20,  paper:60, scissors:20, pwr:50, iq:60,  grit:60, flavor:'Folded 1,000 times.',                      bg:'bg-gradient-to-br from-emerald-500 to-teal-900', rarity:'Epic' },
  { id:'c17', name:'Mantis Blade',    emoji:'🦗', rock:5,   paper:5,  scissors:90, pwr:25, iq:50,  grit:50, flavor:"Nature's perfect cutting machine.",        bg:'bg-gradient-to-br from-lime-600 to-green-900',   rarity:'Common' },
  { id:'c18', name:'Ninja',           emoji:'🥷', rock:10,  paper:10, scissors:80, pwr:35, iq:80,  grit:80, flavor:'Silent, sharp, very pointy.',              bg:'bg-gradient-to-br from-slate-700 to-black',      rarity:'Rare' },
  { id:'c19', name:'Barber Surgeon',  emoji:'💈', rock:15,  paper:15, scissors:70, pwr:20, iq:90,  grit:50, flavor:'Just a little off the top...',             bg:'bg-gradient-to-br from-rose-500 to-red-800',     rarity:'Epic',   passiveId:'SABOTAGE',       passiveDesc:'Randomizes enemy biases.' },
  { id:'c20', name:'Chameleon',       emoji:'🦎', rock:33,  paper:34, scissors:33, pwr:10, iq:70,  grit:80, flavor:'Adapts to anything.',                      bg:'bg-gradient-to-br from-green-400 to-blue-500',   rarity:'Rare',   passiveId:'MIRROR',         passiveDesc:'Copies enemy stats.' },
  { id:'c21', name:'Cursed Doll',     emoji:'🎎', rock:40,  paper:40, scissors:20, pwr:60, iq:10,  grit:90, flavor:'It stares into your soul.',                bg:'bg-gradient-to-br from-purple-800 to-black',     rarity:'Rare',   passiveId:'CURSE_TRAITOR',  passiveDesc:'20% chance to force loss.' },
  { id:'c22', name:'Bulldozer',       emoji:'🚜', rock:95,  paper:5,  scissors:0,  pwr:45, iq:10,  grit:70, flavor:'Paves over the competition.',              bg:'bg-gradient-to-br from-yellow-500 to-yellow-800',rarity:'Rare' },
  { id:'c23', name:'Kite Fighter',    emoji:'🪁', rock:0,   paper:80, scissors:20, pwr:15, iq:60,  grit:30, flavor:'Dances on the wind.',                      bg:'bg-gradient-to-br from-sky-300 to-indigo-600',   rarity:'Common' },
  { id:'c24', name:'Pirate Crab',     emoji:'🏴‍☠️', rock:20, paper:10, scissors:70, pwr:30, iq:45,  grit:65, flavor:'Avast ye!',                              bg:'bg-gradient-to-br from-red-800 to-black',        rarity:'Rare',   passiveId:'COIN_FARMER',    passiveDesc:'+15 coins on Arena win.' },
  { id:'c25', name:'Fortune Teller',  emoji:'🔮', rock:33,  paper:33, scissors:34, pwr:5,  iq:99,  grit:20, flavor:'I see a loss in your future.',             bg:'bg-gradient-to-br from-fuchsia-300 to-purple-600',rarity:'Epic',   passiveId:'SCOUTER',        passiveDesc:'Reveals hidden stats.' }
]);

const BOSS_CARDS = Object.freeze([
  { id:'b1',    name:'Mecha-Rhino',         emoji:'🦾', rock:95,  paper:5,  scissors:0,  pwr:80,  iq:40,  grit:95,  flavor:'Iron-clad engine of destruction.', bg:'bg-gradient-to-br from-red-900 to-black',    rarity:'Boss', passiveId:'COIN_FARMER', passiveDesc:'+15 coins on Arena win.' },
  { id:'b2',    name:'Oracle Tortoise',     emoji:'🐢', rock:33,  paper:33, scissors:34, pwr:70,  iq:120, grit:50,  flavor:'It knew you were going to read this.', bg:'bg-gradient-to-br from-purple-900 to-black', rarity:'Boss', passiveId:'MIRROR',      passiveDesc:'Scrambles your UI buttons!' },
  { id:'b3',    name:'Gambler Fox',         emoji:'🦊', rock:20,  paper:20, scissors:60, pwr:60,  iq:70,  grit:110, flavor:'The house always wins.',           bg:'bg-gradient-to-br from-yellow-700 to-black', rarity:'Boss', passiveId:'GREED',       passiveDesc:'Multiplies coins by 1.5x.' },
  { id:'b4',    name:'The Monolith',        emoji:'🌋', rock:100, paper:0,  scissors:0,  pwr:150, iq:20,  grit:100, flavor:'An ancient, immovable object.',    bg:'bg-gradient-to-br from-stone-800 to-black',  rarity:'Boss', passiveId:'BLOCK_S',     passiveDesc:'Enemy cannot throw Scissors.' },
  { id:'b5',    name:'Phantom Illusionist', emoji:'🎭', rock:33,  paper:34, scissors:33, pwr:50,  iq:150, grit:80,  flavor:"Now you see it, now you don't.",   bg:'bg-gradient-to-br from-indigo-900 to-black', rarity:'Boss', passiveId:'SABOTAGE',    passiveDesc:'Randomizes your biases.' },
  { id:'b6',    name:'Executioner',         emoji:'⚔️', rock:10,  paper:10, scissors:80, pwr:90,  iq:80,  grit:80,  flavor:'One clean cut.',                   bg:'bg-gradient-to-br from-red-950 to-black',    rarity:'Boss', passiveId:'WIN_TIES',    passiveDesc:'Auto-wins Ties.' },
  { id:'glitch',name:'MissingNo',           emoji:'👾', rock:25,  paper:25, scissors:25, nova:25, pwr:99, iq:100, grit:100, flavor:'E̸R̷R̸O̷R̸',              bg:'bg-gradient-to-br from-gray-900 to-black',   rarity:'Boss', passiveId:'CHAOS_AURA',  passiveDesc:'Randomizes stats every round.' }
]);

const WARDEN_BOSS = Object.freeze({ id:'warden', name:'The Warden', emoji:'🚨', rock:33, paper:33, scissors:34, pwr:120, iq:150, grit:100, flavor:'The fun stops here.', bg:'bg-gradient-to-br from-blue-900 to-black', rarity:'Boss', passiveId:'NO_TRICKS', passiveDesc:'Disables Bluffs & Cheats.' });

const ARENA_RIVALS = Object.freeze([
  [{ id:'r1',name:"Rookie Joey",    avatar:"🧑‍🎤",color:"text-green-400", trapMod:0,    iqBuff:0,   desc:"Predictable. Low trap chance." },
   { id:'r5',name:"Crusher Craig",  avatar:"👷‍♂️",color:"text-yellow-500",trapMod:0,    iqBuff:-20, desc:"Brute force. Slow reactions." }],
  [{ id:'r2',name:"Hacker Lily",    avatar:"👩‍💻",color:"text-cyan-400",   trapMod:0.25, iqBuff:30,  desc:"High IQ. Swaps fighters often." },
   { id:'r6',name:"Madame Zarina",  avatar:"🧝‍♀️",color:"text-fuchsia-400",trapMod:0.1,  iqBuff:50,  desc:"Clairvoyant. Reads your mind." }],
  [{ id:'r3',name:"Don Silencio",   avatar:"🕴️", color:"text-purple-400", trapMod:0.1,  suspBuff:30,desc:"Bribes Judges. +30 Suspicion." },
   { id:'r7',name:"The Collector",  avatar:"🎩", color:"text-amber-300",  trapMod:0.2,  immuneBluff:true, desc:"Immune to Bluffs." }],
  [{ id:'r4',name:"The Grandmaster",avatar:"🧙‍♂️",color:"text-red-500",    trapMod:0.15, immuneBluff:true, iqBuff:40, desc:"Uses Bosses. Unforgiving AI." }]
]);

const LOCATIONS = Object.freeze([
  { id:'l1',   name:'Boulder Peak',   icon:'⛰️', bias:'R',   desc:'Easy: Rock bias.',                 cost:5,    difficulty:1 },
  { id:'l2',   name:'Origami Forest', icon:'🌲', bias:'P',   desc:'Easy: Paper bias.',                cost:5,    difficulty:1 },
  { id:'l3',   name:'Barber Alley',   icon:'💈', bias:'S',   desc:'Easy: Scissor bias.',              cost:5,    difficulty:1 },
  { id:'l4',   name:'The Casino',     icon:'🎰', bias:'ANY', desc:'Med: Hidden Stats.',               cost:15,   difficulty:2 },
  { id:'l5',   name:'Neon Dojo',      icon:'🏯', bias:'ANY', desc:'Hard: Hidden Stats. High reward.', cost:500,  difficulty:3 },
  { id:'l6',   name:'Alien Crater',   icon:'🛸', bias:'ANY', desc:'Extreme: Bosses & Mutants.',       cost:1200, difficulty:3 },
  { id:'GACHA',name:'Card Shop',      icon:'🛍️', bias:'SHOP',desc:'Buy Booster Packs!',              cost:0,    difficulty:0 },
  { id:'SHOP', name:'The Back Alley', icon:'🕳️', bias:'SHOP',desc:'Buy Black Market Relics & Tricks.',cost:0,   difficulty:0 },
  { id:'GYM',  name:'Iron Dojo',      icon:'🏋️', bias:'GYM', desc:'Pay coins to train cards.',        cost:0,    difficulty:0 }
]);

const FUSION_RECIPES = Object.freeze({
  'RR':{ name:'Titan',     passive:'BLOCK_P',        desc:'Enemy CANNOT throw Paper.' },
  'PP':{ name:'Oracle',    passive:'BLOCK_S',        desc:'Enemy CANNOT throw Scissors.' },
  'SS':{ name:'Assassin',  passive:'BLOCK_R',        desc:'Enemy CANNOT throw Rock.' },
  'PR':{ name:'Observer',  passive:'SCOUTER',        desc:'Reveals hidden enemy stats.' },
  'PS':{ name:'Trickster', passive:'TRICKSTER_SWAP', desc:'Swaps cards with the enemy.' },
  'RS':{ name:'Berserker', passive:'SECOND_CHANCE',  desc:'50% chance to reroll loss.' }
});

const CURSED_POWERS = Object.freeze([
  { id:'CURSE_FRAGILE', desc:'CURSED: Shatters on Ties.' },
  { id:'CURSE_BLEED',   desc:'CURSED: Bleeds 10 coins.' },
  { id:'CURSE_TRAITOR', desc:'CURSED: 20% chance to force loss.' }
]);

const ARENA_MODIFIERS = Object.freeze([
  { id:'NORMAL',   name:'Standard Rules',        color:'bg-slate-700 text-white' },
  { id:'ROCK_FEST',name:'Rock Festival (+10🪙)',  color:'bg-stone-600 text-white' },
  { id:'BLIND',    name:'Lights Out (Hidden)',    color:'bg-purple-900 text-purple-200' },
  { id:'BAN_S',    name:'Scissor Ban',            color:'bg-red-800 text-red-100' },
  { id:'CHAOS',    name:'Chaos Theory',           color:'bg-yellow-500 text-black' }
]);

const RELICS_DB = Object.freeze([
  { id:'r1', name:'Brass Knuckles',   icon:'🥊', cost:1000, desc:'+15 Power in all Clashes.' },
  { id:'r2', name:'Bribe Money',      icon:'💼', cost:1500, desc:'Cheating base Suspicion is only 25%.' },
  { id:'r3', name:'Vampire Fangs',    icon:'🦇', cost:2000, desc:'Heal 5% of Pot on Match win.' },
  { id:'r4', name:'Clean Record',     icon:'📜', cost:2500, desc:'Cheat Disabled. +40 Clash PWR.' },
  { id:'r5', name:'Stellar Catalyst', icon:'🌌', cost:3000, desc:'Nova unlocks at Level 3.' },
  { id:'r6', name:'Loaded Dice',      icon:'🎲', cost:1500, desc:'Start runs at 20% Suspicion, x2 Pot.' }
]);

const TRICKS_DB = Object.freeze([
  { id:'t1', name:'Smoke Bomb',   icon:'💨', cost:300, desc:'Max Reaction Time (1.5s).' },
  { id:'t2', name:'Energy Drink', icon:'☕', cost:400, desc:'+50 Clash PWR.' },
  { id:'t3', name:'X-Ray Specs',  icon:'🔍', cost:500, desc:'100% Read Chance.' }
]);

const TOURNAMENT_STAGES = Object.freeze([
  { round:1, name:"QUALIFIERS",     mult:2,  modOverride:'NORMAL',       hideStats:false, isBoss:false },
  { round:2, name:"QUARTER-FINALS", mult:4,  modOverride:null,           hideStats:false, isBoss:false },
  { round:3, name:"SEMI-FINALS",    mult:8,  modOverride:'BLIND',        hideStats:true,  isBoss:false },
  { round:4, name:"GRAND FINALS",   mult:20, modOverride:'SUDDEN_DEATH', hideStats:true,  isBoss:true  }
]);

const SYNERGIES = Object.freeze({
  MONOLITH: { name:'The Monolith',     desc:'3 Rock: Immune to Sabotage, Jamming & Judges.', color:'text-stone-400 border-stone-600 bg-stone-900' },
  SWARM:    { name:'The Swarm',        desc:'3 Paper: Enemy stats always revealed.',          color:'text-emerald-400 border-emerald-600 bg-emerald-900' },
  ASSASSINS:{ name:'The Assassins',    desc:'3 Scissors: Auto-win vs Paper.',                 color:'text-orange-400 border-orange-600 bg-orange-900' },
  CHIMERAS: { name:'Lab Rats',         desc:'3 Mutants: Immune to AI Traps.',                 color:'text-pink-400 border-pink-600 bg-pink-900' },
  KINGS:    { name:'Underground Kings',desc:'3 Bosses: Cheating adds 0 Suspicion.',           color:'text-red-400 border-red-600 bg-red-900' }
});

const SYNDICATE_UPGRADES = Object.freeze([
  { id:'hustler',  name:'Street Hustler',  desc:'Start runs with +100 Coins per level.', max:5, cost:5 },
  { id:'shadow',   name:'Shadow Protocol', desc:'Suspicion gains reduced by 5% per level.', max:5, cost:10 },
  { id:'scientist',name:'Mad Scientist',   desc:'Fusion Curse risk reduced by 5% per level.', max:4, cost:15 }
]);

const CITY_EVENTS = Object.freeze([
  { id:'NORMAL',    name:'City Calm',       desc:'Normal conditions.',                    effect:null,          color:'text-slate-400' },
  { id:'SALE',      name:'Midnight Market', desc:'Card Shop packs are 50% off.',          effect:'half_gacha',  color:'text-fuchsia-400' },
  { id:'CRACKDOWN', name:'Police Crackdown',desc:'Wanted Level rises 2x faster.',         effect:'double_heat', color:'text-red-500' },
  { id:'BLOODMOON', name:'Blood Moon',      desc:'Mutations guarantee Epic, 50% Cursed.', effect:'cursed_fusion',color:'text-rose-600' }
]);

// ── Singleton AudioContext ────────────────────────────────────────────────────
let _audioCtx = null;
const getAudioCtx = () => {
  if (!_audioCtx) {
    const AC = window.AudioContext || window.webkitAudioContext;
    if (AC) _audioCtx = new AC();
  }
  return _audioCtx;
};

// Fallback synthesizer for quick UI sounds
const playSynthSound = (type) => {
  try {
    const ctx = getAudioCtx(); if (!ctx) return;
    if (ctx.state === 'suspended') ctx.resume();
    const osc = ctx.createOscillator(), gain = ctx.createGain();
    osc.connect(gain); gain.connect(ctx.destination);
    const t = ctx.currentTime;
    const cfgs = {
      blip:  ()=>{ osc.type='sine';     osc.frequency.setValueAtTime(800,t); osc.frequency.exponentialRampToValueAtTime(1200,t+.05); gain.gain.setValueAtTime(.2,t); gain.gain.exponentialRampToValueAtTime(.01,t+.05); osc.start(t); osc.stop(t+.05); },
      equip: ()=>{ osc.type='square';   osc.frequency.setValueAtTime(400,t); osc.frequency.setValueAtTime(600,t+.05); gain.gain.setValueAtTime(.1,t); gain.gain.linearRampToValueAtTime(.01,t+.1); osc.start(t); osc.stop(t+.1); }
    };
    cfgs[type]?.();
  } catch(_) {}
};

// ── Pure helpers ──────────────────────────────────────────────────────────────
const getDominantStat = (c) => {
  if (!c) return 'R';
  const m = Math.max(c.rock, c.paper, c.scissors, c.nova||0);
  if (m < 40 && m !== c.nova) return '?';
  return m===c.rock?'R':m===c.paper?'P':m===c.scissors?'S':'N';
};

const rollMoveFiltered = (card, blocked=[], force=null) => {
  if (force) return force;
  let r=card?.rock??33, p=card?.paper??33, s=card?.scissors??33, n=card?.nova??0;
  if (blocked.includes('R')) r=0; if (blocked.includes('P')) p=0; if (blocked.includes('S')) s=0;
  if (r>=100&&!blocked.includes('R')) return 'R';
  if (p>=100&&!blocked.includes('P')) return 'P';
  if (s>=100&&!blocked.includes('S')) return 'S';
  const total=r+p+s+n;
  if (total<=0){ const av=['R','P','S'].filter(m=>!blocked.includes(m)); return av.length?av[Math.floor(Math.random()*av.length)]:'R'; }
  const roll=Math.random()*total;
  if (roll<r) return 'R'; if (roll<r+p) return 'P'; if (roll<r+p+s) return 'S'; return 'N';
};

const evaluateMatch = (m1,m2) => {
  if (m1===m2) return 0;
  if (m1==='N') return Math.random()<.70?1:-1;
  if (m2==='N') return Math.random()<.70?-1:1;
  return (m1==='R'&&m2==='S')||(m1==='P'&&m2==='R')||(m1==='S'&&m2==='P')?1:-1;
};

const getCounterMove = m => m==='R'?'P':m==='P'?'S':'R';
const getDupsReq     = lvl => lvl;

const getLeveledCard = (base, level, heat, relics=[]) => {
  if (!base) return null;
  let r=base.rock, p=base.paper, s=base.scissors, n=0;
  let pwr=base.pwr, iq=base.iq||10, grit=base.grit||10;
  if (base.id==='glitch'&&base.passiveId==='CHAOS_AURA') {
    const p1=Math.floor(Math.random()*100), p2=Math.floor(Math.random()*(100-p1)), p3=100-p1-p2;
    [r,p,s]=[p1,p2,p3].sort(()=>Math.random()-.5); n=Math.random()<.2?20:0;
  } else if (level>1) {
    const maxK=Object.keys({R:r,P:p,S:s}).reduce((a,b)=>base[a==='R'?'rock':a==='P'?'paper':'scissors']>base[b==='R'?'rock':b==='P'?'paper':'scissors']?a:b);
    const b=(level-1)*6;
    if(maxK==='R'){r+=b;p-=b/2;s-=b/2}else if(maxK==='P'){p+=b;r-=b/2;s-=b/2}else{s+=b;r-=b/2;p-=b/2}
    r=Math.max(0,r);p=Math.max(0,p);s=Math.max(0,s);
    const tot=r+p+s; r=(r/tot)*100; p=(p/tot)*100; s=(s/tot)*100;
  }
  if (level>=4||(level>=3&&relics.includes('r5'))) { if(base.id!=='glitch'){n=15;r*=.85;p*=.85;s*=.85;} }
  if (base.id!=='glitch') {
    let rR=Math.round(r),rP=Math.round(p),rS=Math.round(s);
    const diff=(100-n)-(rR+rP+rS);
    if(diff>0)rR+=diff; else if(diff<0){if(rR>0)rR+=diff;else if(rP>0)rP+=diff;else rS+=diff;}
    r=Math.max(0,rR);p=Math.max(0,rP);s=Math.max(0,rS);
  }
  if(heat>=3)pwr+=50; if(heat>=5){pwr+=80;iq+=20;grit+=20;} if(level>=10)pwr+=100;
  return {...base,level,rock:r,paper:p,scissors:s,nova:n,pwr,iq,grit,heat:heat||0};
};

const battleReducer = (state, action) => {
  switch (action.type) {
    case 'INIT':  return action.payload;
    case 'RESET': return null;
    case 'FN':    return action.fn(state);
    default:      return state;
  }
};

const ActionBtn = memo(({ onClick, children, className='', disabled, onPointerDown }) => (
  <button onClick={onClick} onPointerDown={onPointerDown} disabled={disabled}
    className={`relative font-black rounded-xl transition-all duration-100 active:top-1 disabled:opacity-50 disabled:cursor-not-allowed disabled:active:top-0 hover:brightness-110 flex items-center justify-center ${className}`}>
    <div className="absolute inset-0 rounded-xl bg-black/40 -bottom-1.5 pointer-events-none"/>
    <div className={`relative h-full w-full flex items-center justify-center rounded-xl border border-white/20 px-4 py-3 pointer-events-none`}>{children}</div>
  </button>
));

const CardBack = memo(() => (
  <div className="w-full aspect-[2/3] max-w-[150px] bg-gradient-to-br from-indigo-700 to-purple-900 border-4 border-indigo-400 rounded-2xl flex items-center justify-center shadow-[0_0_20px_rgba(79,70,229,0.8)] cursor-pointer hover:scale-105 transition-transform animate-pulse">
    <div className="text-4xl">❓</div>
  </div>
));

const CardView = memo(({ card, onClick, selected, small=false, copies=0, dupsReq=1, hideStats=false, isFighting=false, isLabDisabled=false, sellMode=false, equipped=false }) => {
  if (!card) return (
    <div className="w-full aspect-[2/3] border-2 border-dashed border-slate-600 rounded-2xl flex items-center justify-center bg-slate-800/40 cursor-pointer hover:bg-slate-700/50 transition-colors shadow-inner" onClick={()=>{ onClick?.(); }}>
      <span className="text-slate-500 text-[10px] font-black uppercase text-center tracking-widest leading-tight">Empty<br/>Slot</span>
    </div>
  );
  const isBoss=card.rarity==='Boss', isCursed=card.passiveId?.startsWith('CURSE_');
  const isAscended=card.level>=10, isHolo=card.level>=5||isBoss||card.heat>=3||isAscended;
  const isOverdrive=card.heat>=3&&card.heat<5, isOverheat=card.heat>=5;
  const domStat=getDominantStat(card);
  const border=sellMode?'border-red-500 shadow-[0_0_20px_rgba(239,68,68,0.8)] animate-pulse'
    :isAscended?'animated-border ring-2 ring-white/50 shadow-[0_0_30px_rgba(34,211,238,0.8)]'
    :isCursed?'border-red-900 shadow-[0_0_15px_rgba(153,27,27,0.8)] ring-2 ring-red-500'
    :isOverheat?'border-red-500 shadow-[0_0_30px_rgba(239,68,68,1)] ring-2 ring-red-400'
    :isOverdrive?'border-yellow-400 shadow-[0_0_25px_rgba(250,204,21,0.9)] ring-2 ring-orange-500'
    :isBoss?'border-red-600 shadow-[0_0_15px_rgba(220,38,38,0.8)]'
    :card.rarity==='Epic'?'border-purple-500 shadow-lg shadow-purple-500/20'
    :card.rarity==='Rare'?'border-blue-500 shadow-lg shadow-blue-500/20'
    :card.rarity==='Mutant'?'border-pink-500 shadow-lg shadow-pink-500/30'
    :'border-slate-700 shadow-md shadow-black/50';
  const badge=isAscended?'bg-cyan-300 text-black shadow-[0_0_10px_cyan]'
    :isCursed?'bg-red-900 text-red-100 animate-pulse':isBoss?'bg-red-700 text-white'
    :card.rarity==='Epic'?'bg-purple-600 text-white':card.rarity==='Rare'?'bg-blue-600 text-white'
    :card.rarity==='Mutant'?'bg-pink-600 text-white':'bg-slate-700 text-slate-300';
  const tellAnim=isFighting?(domStat==='R'?'animate-[thud_1.5s_infinite_alternate]':domStat==='P'?'animate-[float_2s_infinite_ease-in-out]':'animate-[twitch_0.5s_infinite]'):'';
  const extraAnim=isOverheat?' animate-[twitch_0.2s_infinite]':(isOverdrive||isAscended)?' animate-[float_1s_infinite_ease-in-out]':'';
  const handleClick=()=>{ if(!isLabDisabled&&onClick){ onClick(); } };
  return (
    <div onClick={handleClick}
      className={`w-full aspect-[2/3] relative rounded-2xl flex flex-col items-center justify-between p-1.5 transition-all duration-200 ${card.bg} ${selected&&!sellMode?'ring-4 ring-cyan-400 scale-105 z-10 shadow-[0_0_30px_rgba(34,211,238,0.8)]':''} ${isLabDisabled?'opacity-40 grayscale cursor-not-allowed':onClick?'cursor-pointer hover:-translate-y-1 hover:shadow-2xl':''} border-[3px] ${border} ${tellAnim}${extraAnim} overflow-hidden`}>
      {equipped&&!sellMode&&<div className="absolute top-1 left-1 bg-yellow-400 text-black text-[8px] font-black px-1 rounded z-20 shadow-md">EQUIPPED</div>}
      {isHolo&&!isCursed&&!sellMode&&<div className="absolute top-0 bottom-0 w-[200%] bg-gradient-to-r from-transparent via-white/40 to-transparent z-0 pointer-events-none animate-shimmer"/>}
      {isCursed&&!sellMode&&<div className="absolute inset-0 bg-[repeating-radial-gradient(circle_at_center,transparent_0,transparent_10px,rgba(153,27,27,0.3)_10px,rgba(153,27,27,0.3)_20px)] z-0 pointer-events-none opacity-60"/>}
      {isOverheat&&!sellMode&&<div className="absolute inset-0 bg-red-600/40 animate-pulse z-0 pointer-events-none mix-blend-multiply border-4 border-red-500 rounded-lg"/>}
      {!isOverheat&&isOverdrive&&!sellMode&&<div className="absolute inset-0 bg-yellow-500/20 animate-pulse z-0 pointer-events-none mix-blend-color-dodge border-2 border-yellow-300 rounded-lg"/>}
      {isAscended&&!sellMode&&<div className="absolute inset-0 bg-cyan-400/30 animate-pulse z-0 pointer-events-none mix-blend-screen border-2 border-cyan-200 rounded-lg shadow-[inset_0_0_20px_cyan]"/>}
      {sellMode&&<div className="absolute inset-0 bg-red-900/60 z-20 flex items-center justify-center pointer-events-none"><Trash2 size={40} className="text-red-300 drop-shadow-md opacity-50"/></div>}
      <div className="flex justify-between items-start w-full z-10 mb-0.5">
        <div className={`font-black tracking-tighter leading-none text-shadow-sm ${small?'text-[8px]':'text-xs'} ${isBoss||isCursed||isOverdrive||isAscended?'text-white':'text-gray-900'} truncate flex-1`}>{card.name}</div>
        <div className={`${DNA_COLORS[domStat]} border border-black/50 text-white rounded-full flex items-center justify-center font-black ${small?'w-3 h-3 text-[6px]':'w-4 h-4 text-[8px]'} ml-1 shrink-0 shadow-sm`}>{domStat}</div>
      </div>
      <div className={`w-full bg-white/95 border-2 border-black/30 rounded-md flex items-center justify-center flex-1 z-10 relative shadow-inner ${small?'mb-0.5':'mb-1'} ${isCursed?'bg-red-200':''} ${isOverheat?'bg-red-200 shadow-[inset_0_0_20px_rgba(220,38,38,0.8)]':isOverdrive?'bg-orange-100 shadow-[inset_0_0_15px_rgba(234,88,12,0.5)]':isAscended?'bg-cyan-100 shadow-[inset_0_0_20px_rgba(34,211,238,0.8)]':''}`}>
        {card.level>1&&<div className={`absolute -top-1 -left-1 ${isAscended?'bg-cyan-500 border-cyan-200':card.level>=4?'bg-indigo-600 border-indigo-900':'bg-blue-600 border-blue-900'} text-white font-black rounded-sm border flex items-center justify-center shadow-sm ${small?'text-[8px] px-1 py-0.5 leading-none':'text-[10px] px-1.5 py-0.5'}`}>L{card.level}</div>}
        <div className={`absolute -top-1 -right-1 ${isOverheat?'bg-red-700 animate-bounce':'bg-amber-500'} text-white font-black rounded-full border-2 ${isOverdrive?'border-red-900':'border-amber-800'} flex items-center justify-center shadow-md ${small?'text-[6px] w-[18px] h-[18px] leading-tight':'text-[8px] w-6 h-6 leading-tight'} flex-col`}>
          <span>{isAscended?'✨':isOverheat?'⚠️':isOverdrive?'🔥':'⚡'}</span><span>{card.pwr}</span>
        </div>
        <div className={`${small?'text-2xl':'text-5xl'} drop-shadow-md ${isOverdrive||isAscended?'animate-[bounce_0.5s_infinite]':''}`}>{sellMode?'🗑️':card.emoji}</div>
      </div>
      <div className={`w-full flex flex-col items-center justify-center z-10 ${small?'gap-0.5 mb-0.5':'gap-1 mb-1'}`}>
        <div className="flex gap-1">
          {(card.rarity!=='Common'||isHolo)&&<div className={`${small?'text-[5px]':'text-[7px]'} font-black px-1 rounded uppercase tracking-widest border border-black/30 ${badge}`}>{isAscended?'ASCENDED':isCursed?'CURSED':card.level>=5&&!isBoss?'HOLO MAX':card.rarity}</div>}
          {card.heat>0&&<div className={`${small?'text-[5px]':'text-[7px]'} font-black px-1 rounded ${isOverheat?'bg-red-700 animate-pulse':'bg-orange-600'} text-white flex items-center`}>{Array(Math.min(card.heat,5)).fill('🔥').join('')}</div>}
        </div>
        {card.passiveDesc&&!small&&<div className="text-[8px] leading-[9px] text-center bg-black/75 rounded px-1 w-full py-0.5 flex items-center justify-center font-bold text-white shadow-sm border border-white/10">{card.passiveId&&<span className="text-yellow-400 mr-0.5">{isCursed?'🩸':'✨'}</span>}{card.passiveDesc}</div>}
      </div>
      <div className={`w-full bg-slate-900 rounded-md flex justify-around items-center z-10 text-white font-black border border-white/20 shadow-sm ${small?'px-0.5 py-0.5 text-[6px] mb-0.5':'px-1 py-0.5 text-[8px] mb-1'}`}>
        <span className="text-blue-300 flex items-center gap-0.5"><Brain size={small?8:10}/> {card.iq}</span>
        <span className="text-purple-300 flex items-center gap-0.5"><Glasses size={small?8:10}/> {card.grit}</span>
      </div>
      <div className={`w-full bg-black/95 rounded-md flex justify-between items-center z-10 text-white font-black border border-white/20 shadow-sm ${small?'px-0.5 py-0.5 text-[7px]':'px-1.5 py-1 text-[9px]'}`}>
        <span className={card.rock>50&&!hideStats?'text-cyan-400':'text-gray-400'}>✊{hideStats?'?':Math.round(card.rock)}</span>
        <span className={card.paper>50&&!hideStats?'text-cyan-400':'text-gray-400'}>✋{hideStats?'?':Math.round(card.paper)}</span>
        <span className={card.scissors>50&&!hideStats?'text-cyan-400':'text-gray-400'}>✌️{hideStats?'?':Math.round(card.scissors)}</span>
        {card.nova>0&&<span className="text-indigo-400">🌟{hideStats?'?':card.nova}</span>}
      </div>
      {!small&&card.level<10&&card.rarity!=='Mutant'&&!isBoss&&<div className="absolute bottom-0 left-0 w-full h-1 bg-black/80 z-20"><div className="bg-cyan-400 h-full" style={{width:`${(copies/dupsReq)*100}%`}}/></div>}
      {isLabDisabled&&<div className="absolute inset-0 bg-black/70 flex items-center justify-center z-30"><div className="text-[10px] font-black text-red-500 bg-black px-2 py-1 rounded border-2 border-red-500 uppercase text-center rotate-[-15deg] shadow-lg">Lvl 2+<br/>Req</div></div>}
    </div>
  );
});

const DeckGrid = memo(({ ids, inventoryMap, getCard, onCardClick, teamIds, fusionSlots, sellMode, isLab, deckSort }) => {
  const sorted = useMemo(() => [...ids].sort((a,b) => {
    const cA=getCard(a), cB=getCard(b); if(!cA||!cB) return 0;
    if(deckSort==='LVL') return cB.level-cA.level;
    if(deckSort==='PWR') return cB.pwr-cA.pwr;
    if(deckSort==='ELM') return getDominantStat(cA).localeCompare(getDominantStat(cB));
    return 0;
  }), [ids, deckSort, getCard]);

  if (!sorted.length) return (
    <div className="col-span-3 sm:col-span-4 text-center mt-10 w-full"><p className="text-slate-500 font-black text-xl">NO CARDS MATCH</p></div>
  );
  return (
    <div className="grid grid-cols-3 sm:grid-cols-4 gap-2 sm:gap-3 auto-rows-max items-start perspective-1000 pb-12 pt-2">
      {sorted.map(id => {
        const card=getCard(id);
        return (
          <CardView key={id} card={card}
            selected={isLab?fusionSlots.includes(id):teamIds.includes(id)}
            copies={inventoryMap[id]?.copies??0}
            dupsReq={getDupsReq(inventoryMap[id]?.level??1)}
            onClick={()=>onCardClick(id)}
            isLabDisabled={isLab&&card?.level<2}
            sellMode={sellMode}
            equipped={teamIds.includes(id)}
          />
        );
      })}
    </div>
  );
});


// ── MAIN APP ──────────────────────────────────────────────────────────────────
export default function App() {
  const [appState, setAppState]   = useState('MENU');
  const [gameMode, setGameMode]   = useState(null);
  const [shakeScreen, setShakeScreen] = useState(false);
  const [inputLocked, setInputLocked] = useState(false);
  
  // Game Settings & Progression
  const [isMuted, setIsMuted]     = useState(false);
  const [showTutorial, setShowTutorial] = useState(false);
  const [devClicks, setDevClicks] = useState(0);

  const [trophies, setTrophies]   = useState(0);
  const [upgrades, setUpgrades]   = useState({ hustler: 0, shadow: 0, scientist: 0 });
  const [runSummary, setRunSummary] = useState(null);
  const [discovered, setDiscovered] = useState(() => INITIAL_CARDS.map(c => c.id));

  const [view, setView]           = useState('EXPLORE');
  const [coins, setCoins]         = useState(0);
  const coinsRef = useRef(coins);

  const updateCoins = useCallback((updater) => {
    setCoins(prev => {
      const next = typeof updater === 'function' ? updater(prev) : updater;
      coinsRef.current = next;
      return next;
    });
  }, []);

  const [inventoryMap, setInventoryMap] = useState({});
  const [toast, setToast]         = useState(null);
  const toastTimerRef             = useRef(null);

  const showToast = useCallback((msg, isBad = false) => {
    if (toastTimerRef.current) clearTimeout(toastTimerRef.current);
    setToast({ msg, isBad });
    toastTimerRef.current = setTimeout(() => setToast(null), 3000);
  }, []);

  const triggerShake = useCallback(() => {
    setShakeScreen(true);
    window.navigator?.vibrate?.(200);
    setTimeout(() => setShakeScreen(false), 300);
  }, []);

  const [customCards, setCustomCards] = useState([]);
  const allCards = useMemo(() => [...INITIAL_CARDS, ...BOSS_CARDS, ...customCards], [customCards]);

  const [relics, setRelics]   = useState([]);
  const [tricks, setTricks]   = useState([]);
  const [teamIds, setTeamIds] = useState([null, null, null, null]);
  const [unlockedLocs, setUnlockedLocs] = useState([]);

  const [deckFilter, setDeckFilter]       = useState('ALL');
  const [deckSort, setDeckSort]           = useState('LVL');
  const [sellMode, setSellMode]           = useState(false);
  const [blessedFusion, setBlessedFusion] = useState(false);
  const [showRetireConfirm, setShowRetireConfirm] = useState(false);
  const [inspectedCardId, setInspectedCardId]     = useState(null);
  const [bloodPactOffered, setBloodPactOffered]   = useState(false);
  const [showDarkWeb, setShowDarkWeb]             = useState(false);

  const [encTarget, setEncTarget]   = useState(null);
  const [encLoc, setEncLoc]         = useState(null);
  const [catchRes, setCatchRes]     = useState(null);
  const [isScanned, setIsScanned]   = useState(false);
  const [activeEvent, setActiveEvent] = useState(null);
  const [cityHeat, setCityHeat]     = useState(0);
  const [spamTracker, setSpamTracker] = useState({ move: null, count: 0 });
  const [ambushActive, setAmbushActive] = useState(false);
  const [cityEvent, setCityEvent]   = useState(CITY_EVENTS[0]);

  const [gachaCards, setGachaCards]   = useState([]);
  const [gachaFlipped, setGachaFlipped] = useState([]);
  const [packTorn, setPackTorn]       = useState(false);

  const battleReducerFull = useCallback((state, action) => {
    if (action.type === '_FN') return action.fn(state);
    return battleReducer(state, action);
  }, []);

  const [battleState, dispatchBattleFull] = useReducer(battleReducerFull, null);

  const [enemyTeam, setEnemyTeam]   = useState([]);
  const [wager, setWager]           = useState(100);
  const [suspicion, setSuspicion]   = useState(0);
  const [crowdHype, setCrowdHype]   = useState(0);
  const [isRiot, setIsRiot]         = useState(false);
  const [clashTime, setClashTime]   = useState(0);
  const [pulseSync, setPulseSync]   = useState(false);
  const [clashHitMarker, setClashHitMarker] = useState(null);
  const [fusionSlots, setFusionSlots] = useState([null, null]);
  const [trainTarget, setTrainTarget] = useState(null);

  // ── BGM / Sound Engine ─────────────────────────────────────────────────────
  const bgmRef          = useRef(null);
  const currentTrackRef = useRef(null);
  const audioPoolRef    = useRef({});
  const audioUnlockedRef = useRef(false);

  useEffect(() => {
    const unlockAudio = () => {
      audioUnlockedRef.current = true;
      if (!isMuted && bgmRef.current && bgmRef.current.paused && bgmRef.current.src) {
        bgmRef.current.play().catch(()=>{});
      }
    };
    document.addEventListener('pointerdown', unlockAudio, { once: true });
    document.addEventListener('click', unlockAudio, { once: true });
    return () => {
      document.removeEventListener('pointerdown', unlockAudio);
      document.removeEventListener('click', unlockAudio);
    };
  }, [isMuted]);

  const playSound = useCallback((type) => {
    if (isMuted) return;
    if (AUDIO_ASSETS[type]) {
      try {
        if (type === 'hit') {
           const hitSnd = new Audio(AUDIO_ASSETS[type]);
           hitSnd.volume = 0.3;
           hitSnd.play().catch(()=>{});
        } else {
           if (!audioPoolRef.current[type]) { audioPoolRef.current[type] = new Audio(AUDIO_ASSETS[type]); }
           const snd = audioPoolRef.current[type];
           snd.currentTime = 0; snd.volume = 0.6; snd.play().catch(()=>{});
        }
      } catch (_) {}
    } else {
      playSynthSound(type);
    }
  }, [isMuted]);

  useEffect(() => {
    if (!bgmRef.current) bgmRef.current = Object.assign(new Audio(), { loop: true, volume: 0.3 });
    if (isMuted) { bgmRef.current.pause(); return; }

    let neededTrack = 'bgm_explore';
    if (appState === 'MENU' || appState === 'SYNDICATE' || appState === 'COMPENDIUM') neededTrack = 'bgm_menu';
    else if (appState === 'SUMMARY') neededTrack = runSummary?.bankrupt ? 'bgm_dark' : 'bgm_menu';
    else if (view === 'BATTLE_PLAY') {
       if (isRiot) neededTrack = 'crowd_riot';
       else if (battleState?.phase === 'ULTIMATE_OFFER' || battleState?.isSuddenDeath) neededTrack = 'heartbeat';
       else neededTrack = 'bgm_battle';
    }
    else if (view === 'CATCH') { neededTrack = ambushActive ? 'sirens' : 'bgm_battle'; }
    else if (view === 'BLACK_MARKET' || showDarkWeb) neededTrack = 'bgm_dark';

    const src = AUDIO_ASSETS[neededTrack];
    if (!src) return;
    if (currentTrackRef.current !== neededTrack) {
      currentTrackRef.current = neededTrack;
      bgmRef.current.src = src;
      if (audioUnlockedRef.current) bgmRef.current.play().catch(() => {});
    } else if (bgmRef.current.paused && audioUnlockedRef.current) {
      bgmRef.current.play().catch(() => {});
    }
  }, [view, appState, isMuted, isRiot, showDarkWeb, ambushActive, runSummary, battleState?.phase, battleState?.isSuddenDeath]);


  const maxSuspicion = useMemo(() => relics.includes('r6') ? 150 : 100, [relics]);
  const suspRate     = useMemo(() => relics.includes('r2') ? 25 : 40, [relics]);

  const getCard = useCallback((id) => {
    if (!id) return null;
    const base = allCards.find(c => c.id === id);
    if (!base) return null;
    const inv = inventoryMap[id];
    return getLeveledCard(base, inv?.level || 1, inv?.heat || 0, relics);
  }, [allCards, inventoryMap, relics]);

  const teamScout    = useMemo(() => teamIds.some(id => id && getCard(id)?.passiveId === 'SCOUTER'), [teamIds, getCard]);
  const teamGreed    = useMemo(() => teamIds.some(id => id && getCard(id)?.passiveId === 'GREED'), [teamIds, getCard]);

  const activeSynergy = useMemo(() => {
    const cards = teamIds.slice(0, 3).map(id => id ? getCard(id) : null);
    if (cards.some(c => !c)) return null;
    if (cards.every(c => c.rarity === 'Mutant')) return 'CHIMERAS';
    if (cards.every(c => c.rarity === 'Boss'))   return 'KINGS';
    const doms = cards.map(getDominantStat);
    if (doms.every(d => d === 'R')) return 'MONOLITH';
    if (doms.every(d => d === 'P')) return 'SWARM';
    if (doms.every(d => d === 'S')) return 'ASSASSINS';
    return null;
  }, [teamIds, getCard]);

  const safeAction = useCallback((fn) => (e) => {
    if (inputLocked) { e?.preventDefault(); return; }
    playSound('blip');
    fn(e);
  }, [inputLocked, playSound]);

  const filteredDeckIds = useMemo(() => {
    return Object.keys(inventoryMap).filter(id => {
      if (deckFilter === 'ALL') return true;
      const c = getCard(id);
      return c && getDominantStat(c) === deckFilter;
    });
  }, [inventoryMap, deckFilter, getCard]);

  const markDiscovered = useCallback((id) => {
    setDiscovered(prev => prev.includes(id) ? prev : [...prev, id]);
  }, []);

  const handleDevClick = useCallback(() => {
    setDevClicks(c => c + 1);
  }, []);

  useEffect(() => {
    if (devClicks >= 5) {
      setTimeout(() => {
        updateCoins(c => c + 100000);
        showToast('DEV MODE: 100k COINS GRANTED!', false);
        playSound('vocal_win');
      }, 0);
      setDevClicks(0);
    }
  }, [devClicks, updateCoins, playSound, showToast]);


  // ── RUN MANAGEMENT ─────────────────────────────────────────────────────────
  const startRun = useCallback((mode) => {
    playSound('new_game');
    setGameMode(mode);
    if (mode === 'SANDBOX') {
      updateCoins(99999);
      const map = {};
      INITIAL_CARDS.forEach(c => { map[c.id] = { copies: 0, level: 4, heat: 0 }; });
      setInventoryMap(map);
      setUnlockedLocs(['l1','l2','l3','l4','l5','l6','SHOP','GYM','GACHA']);
    } else {
      updateCoins(300 + (upgrades.hustler * 100));
      setInventoryMap({
        'c1': { copies: 0, level: 1, heat: 0 },
        'c2': { copies: 0, level: 1, heat: 0 },
        'c3': { copies: 0, level: 1, heat: 0 }
      });
      setUnlockedLocs(['l1','l2','l3','GACHA','SHOP','GYM']);
    }
    setTeamIds([null,null,null,null]);
    setCustomCards([]);
    setRelics([]);
    setTricks([]);
    setSuspicion(0);
    setCrowdHype(0);
    setIsRiot(false);
    setCityHeat(0);
    setSpamTracker({ move: null, count: 0 });
    dispatchBattleFull({ type: 'RESET' });
    setBloodPactOffered(false);
    setCityEvent(CITY_EVENTS[Math.floor(Math.random() * CITY_EVENTS.length)]);
    setAppState('PLAYING');
    setView('EXPLORE');
  }, [upgrades.hustler, updateCoins, playSound]);

  const endRun = useCallback((bankrupt = false) => {
    if (bankrupt) playSound('game_over'); else playSound('vocal_win');
    const cur = coinsRef.current;
    let earned = bankrupt ? 0 : Math.floor(cur / 1000);
    let bossCount = 0, ascendedCount = 0;
    Object.keys(inventoryMap).forEach(id => {
      const card = getCard(id);
      if (!card) return;
      if (card.rarity === 'Boss') bossCount++;
      if (card.level >= 10) ascendedCount++;
    });
    earned += bossCount * 5 + ascendedCount * 3;
    const finalEarned = Math.max(bankrupt ? 0 : 1, earned);
    setRunSummary({ bankrupt, coins: bankrupt ? 0 : cur, bosses: bossCount, ascended: ascendedCount, trophies: finalEarned });
    setTrophies(t => t + finalEarned);
    setShowRetireConfirm(false);
    setAppState('SUMMARY');
  }, [inventoryMap, getCard, playSound]);

  const buyUpgrade = useCallback((upg) => {
    const currentLvl = upgrades[upg.id];
    if (currentLvl >= upg.max) return;
    if (trophies >= upg.cost) {
      setTrophies(t => t - upg.cost);
      setUpgrades(prev => ({ ...prev, [upg.id]: prev[upg.id] + 1 }));
    } else {
      alert('Not enough Trophies 🏆!');
    }
  }, [upgrades, trophies]);

  const acceptBloodPact = useCallback(() => {
    updateCoins(1500);
    setBloodPactOffered(false);
    dispatchBattleFull({ type: 'RESET' });
    const invKeys = Object.keys(inventoryMap);
    const cursedKeys = [...invKeys].sort(() => 0.5 - Math.random()).slice(0, 3);
    setInventoryMap(prev => {
      const next = { ...prev };
      cursedKeys.forEach(k => {
        if (!next[k]) return;
        const base = getCard(k);
        if (!base?.id.startsWith('c') && !base?.id.startsWith('b')) return;
        const newId  = `cursed_${k}_${Date.now()}`;
        const curse  = CURSED_POWERS[Math.floor(Math.random() * CURSED_POWERS.length)];
        const cursed = { ...base, id: newId, name: `Cursed ${base.name}`, rarity: 'Mutant', bg: 'bg-gradient-to-br from-red-900 to-black', passiveId: curse.id, passiveDesc: curse.desc };
        setCustomCards(p => [...p, cursed]);
        next[newId] = next[k];
        delete next[k];
        setTeamIds(team => team.map(tid => tid === k ? newId : tid));
      });
      return next;
    });
    showToast('BLOOD PACT SEALED. 1500 🪙 GRANTED. CURSES APPLIED.', true);
    triggerShake();
    setView('DECK');
  }, [inventoryMap, getCard, updateCoins, showToast, triggerShake]);

  const toggleTeam = useCallback((id) => {
    setTeamIds(prev => {
      const next = [...prev];
      const idx = next.indexOf(id);
      if (idx > -1) { next[idx] = null; }
      else {
        const empty = next.indexOf(null);
        if (empty > -1) next[empty] = id;
        else showToast('Team is full! Tap an equipped card to remove.', true);
      }
      return next;
    });
  }, [showToast]);

  const handleDeckCardClick = useCallback((id) => {
    if (sellMode) {
      if (teamIds.includes(id)) return showToast('Cannot scrap equipped cards!', true);
      const card = getCard(id);
      if (card?.rarity === 'Boss') return showToast('Bosses cannot be scrapped!', true);
      const val = (card.rarity==='Epic' ? 80 : card.rarity==='Rare' ? 40 : card.rarity==='Mutant' ? 100 : 15) * card.level;
      updateCoins(c => c + val);
      setFusionSlots(prev => [prev[0]===id ? null : prev[0], prev[1]===id ? null : prev[1]]);
      setTrainTarget(prev => prev === id ? null : prev);
      setInventoryMap(prev => { const next = {...prev}; delete next[id]; return next; });
      if (id.startsWith('m_') || id.startsWith('cursed_')) setCustomCards(prev => prev.filter(c => c.id !== id));
      showToast(`Scrapped ${card.name} for ${val} 🪙`);
    } else {
      setInspectedCardId(id);
    }
  }, [sellMode, teamIds, getCard, updateCoins, showToast]);

  const equipFromInspect = useCallback((slotIdx) => {
    if (!inspectedCardId) return;
    playSound('equip');
    setTeamIds(prev => {
      const nt = [...prev];
      const existingIdx = nt.indexOf(inspectedCardId);
      if (existingIdx > -1) nt[existingIdx] = null;
      nt[slotIdx] = inspectedCardId;
      return nt;
    });
    showToast('Equipped!');
  }, [inspectedCardId, playSound, showToast]);

  const handleNav = useCallback((newView) => {
    if (view === 'BATTLE_PLAY' && battleState && battleState.phase !== 'DONE' && battleState.phase !== 'BUSTED') {
      showToast('Cannot leave active combat!', true);
      return false;
    }
    setSellMode(false);
    setDeckFilter('ALL');
    setView(newView);
    return true;
  }, [view, battleState, showToast]);

  // ── FUSION ─────────────────────────────────────────────────────────────────
  const [c1Preview, c2Preview] = useMemo(() => [
    fusionSlots[0] ? getCard(fusionSlots[0]) : null,
    fusionSlots[1] ? getCard(fusionSlots[1]) : null
  ], [fusionSlots, getCard]);

  const predictedRecipe = useMemo(() => {
    if (!c1Preview || !c2Preview) return null;
    const rCombo = [getDominantStat(c1Preview), getDominantStat(c2Preview)].sort().join('');
    return FUSION_RECIPES[rCombo] || { name: 'Amalgam', desc: 'Unknown Wildcard Mutation.' };
  }, [c1Preview, c2Preview]);

  const handleFuse = useCallback(() => {
    if (!fusionSlots[0] || !fusionSlots[1]) return showToast('Select two cards!', true);
    if (coins < 100) return showToast('Need 100 🪙', true);
    const c1 = getCard(fusionSlots[0]), c2 = getCard(fusionSlots[1]);
    if (!c1 || !c2) return;
    if (c1.level < 2 || c2.level < 2) return showToast('Both cards must be Level 2+!', true);
    if (c1.rarity === 'Boss' || c2.rarity === 'Boss') return showToast('Cannot fuse Bosses!', true);
    updateCoins(c => c - 100);

    const baseCurseChance = cityEvent.id === 'BLOODMOON' ? 0.50 : 0.20;
    const curseChance = Math.max(0, baseCurseChance - (upgrades.scientist * 0.05));
    const isCursed = blessedFusion ? false : Math.random() < curseChance;
    if (blessedFusion) setBlessedFusion(false);

    let fR, fP, fS, pId, pDesc, flavor;
    const d1 = getDominantStat(c1), d2 = getDominantStat(c2);
    const combo  = [d1, d2].sort().join('');
    const recipe = FUSION_RECIPES[combo] || { name: 'Amalgam', passive: 'OMNI_WIN', desc: '10% chance to insta-win.' };

    if (isCursed) {
      const curse = CURSED_POWERS[Math.floor(Math.random() * CURSED_POWERS.length)];
      pId = curse.id; pDesc = curse.desc; flavor = 'Catastrophic failure.';
      fR = Math.floor(Math.random() * 100); fP = Math.floor(Math.random() * (100 - fR)); fS = 100 - fR - fP;
      showToast('CATASTROPHIC MUTATION! 🩸', true); triggerShake();
    } else {
      const rAvg = (c1.rock+c2.rock)/2, pAvg = (c1.paper+c2.paper)/2, sAvg = (c1.scissors+c2.scissors)/2;
      const total = rAvg + pAvg + sAvg;
      fR = Math.round((rAvg/total)*100); fP = Math.round((pAvg/total)*100); fS = Math.round((sAvg/total)*100);
      const diff = 100 - (fR+fP+fS);
      if (diff > 0) fR += diff; else if (diff < 0) { if (fR > 0) fR += diff; else if (fP > 0) fP += diff; else fS += diff; }
      pId = recipe.passive; pDesc = recipe.desc; flavor = `${recipe.name} Class Mutant.`;
      showToast(`${recipe.name} Chimera created!`);
    }

    const isEpicBase = cityEvent.id === 'BLOODMOON';
    const n = {
      id: `m_${Date.now()}`, name: `${c1.name.split(' ')[0]} ${c2.name.split(' ')[1] || c2.name}`,
      emoji: MUTANT_EMOJIS[Math.floor(Math.random() * MUTANT_EMOJIS.length)],
      rock: fR, paper: fP, scissors: fS,
      pwr: Math.round((c1.pwr+c2.pwr) * (isEpicBase ? 1.5 : 0.8)),
      iq: Math.max(10, Math.round((c1.iq+c2.iq)/2) - (isCursed ? 20 : 0) + (isEpicBase ? 30 : 0)),
      grit: Math.max(10, Math.round((c1.grit+c2.grit)/2) + (isCursed ? 20 : 0) + (isEpicBase ? 30 : 0)),
      flavor, bg: isCursed ? 'bg-gradient-to-br from-red-700 to-red-950' : 'bg-gradient-to-br from-pink-500 to-purple-800',
      rarity: 'Mutant', passiveId: pId, passiveDesc: pDesc
    };
    setCustomCards(p => [...p, n]);
    setTeamIds(prev => prev.map(id => (id===c1.id || id===c2.id) ? null : id));
    
    setInventoryMap(prev => {
      const next = { ...prev };
      delete next[c1.id];
      delete next[c2.id];
      next[n.id] = { copies:0, level:1, heat:0 };
      return next;
    });
    
    setFusionSlots([null,null]);
    setView('DECK');
  }, [fusionSlots, coins, getCard, cityEvent, upgrades.scientist, blessedFusion, updateCoins, showToast, triggerShake]);

  const togLab = useCallback((id) => {
    setFusionSlots(s => s[0]===id ? [null,s[1]] : s[1]===id ? [s[0],null] : !s[0] ? [id,s[1]] : !s[1] ? [s[0],id] : s);
  }, []);

  // ── GACHA ──────────────────────────────────────────────────────────────────
  const buyPack = useCallback((type) => {
    const cost = type === 'PREMIUM' ? (cityEvent.id==='SALE' ? 750 : 1500) : (cityEvent.id==='SALE' ? 250 : 500);
    if (coins < cost) return showToast('Not enough coins!', true);
    updateCoins(c => c - cost);

    const pool = INITIAL_CARDS.filter(c => c.rarity !== 'Boss');
    const pullCard = (guaranteeEpic) => {
      if (type==='PREMIUM' && Math.random() < 0.01) return BOSS_CARDS.find(b => b.id==='glitch');
      if (guaranteeEpic) return pool.find(c => c.rarity==='Epic');
      const roll = Math.random();
      const epics   = pool.filter(c => c.rarity==='Epic');
      const rares   = pool.filter(c => c.rarity==='Rare');
      const commons = pool.filter(c => c.rarity==='Common');
      if (roll < 0.05) return epics[Math.floor(Math.random()*epics.length)] || pool[0];
      if (roll < 0.30) return rares[Math.floor(Math.random()*rares.length)] || pool[0];
      return commons[Math.floor(Math.random()*commons.length)] || pool[0];
    };

    setGachaCards([pullCard(false), pullCard(false), pullCard(type==='PREMIUM')]);
    setGachaFlipped([false,false,false]);
    setPackTorn(false);
    setView('PACK_OPENING');
  }, [cityEvent, coins, updateCoins, showToast]);

  const claimGacha = useCallback(() => {
    const newDisc = new Set(discovered);
    gachaCards.forEach(c => newDisc.add(c.id));
    if (newDisc.size > discovered.length) setDiscovered(Array.from(newDisc));
    setInventoryMap(prev => {
      const next = { ...prev };
      gachaCards.forEach(c => {
        if (!next[c.id]) { next[c.id] = { copies:0, level:1, heat:0 }; }
        else {
          const cur = { ...next[c.id] };
          if (cur.level < 10) {
            cur.copies += 1;
            if (cur.copies >= getDupsReq(cur.level)) { cur.copies -= getDupsReq(cur.level); cur.level += 1; }
          }
          next[c.id] = cur;
        }
      });
      return next;
    });
    setView('GACHA_SHOP');
  }, [gachaCards, discovered]);

  // ── EXPLORE ────────────────────────────────────────────────────────────────
  const triggerAmbush = useCallback(() => {
    playSound('illegal');
    setAmbushActive(true);
    setEncTarget(WARDEN_BOSS);
    setIsScanned(true);
    setCatchRes(null);
    showToast('🚨 WANTED LEVEL MAX: POLICE AMBUSH! 🚨', true);
    triggerShake();
    setView('CATCH');
  }, [showToast, triggerShake, playSound]);

  const generateEncounter = useCallback((loc) => {
    const isBoss = Math.random() < 0.15 && unlockedLocs.length > 4;
    if (isBoss) {
      playSound('boss');
      setEncTarget(BOSS_CARDS[Math.floor(Math.random()*BOSS_CARDS.length)]);
      showToast('⚠️ BOSS AMBUSH! ⚠️', true);
      triggerShake();
    } else {
      let pool = [...INITIAL_CARDS];
      if (loc?.bias !== 'ANY') { const f = pool.filter(c => getDominantStat(c) === loc?.bias); if (f.length > 0) pool = f; }
      setEncTarget(pool[Math.floor(Math.random()*pool.length)]);
    }
    setIsScanned(false);
    setCatchRes(null);
    setView('CATCH');
  }, [unlockedLocs, showToast, triggerShake, playSound]);

  const clickLoc = useCallback((loc) => {
    if (loc.id === 'SHOP')  return setView('BLACK_MARKET');
    if (loc.id === 'GYM')   return setView('GYM');
    if (loc.id === 'GACHA') return setView('GACHA_SHOP');

    if (unlockedLocs.includes(loc.id)) {
      if (coins < loc.cost) return showToast(`Need ${loc.cost} coins to travel here!`, true);
      updateCoins(c => c - loc.cost);
      if (cityHeat >= 100) { triggerAmbush(); return; }

      if (Math.random() < 0.20 && gameMode === 'ROGUELITE') {
        const events = [
          { title:"Underground Sponsor", text:"A shady figure offers you a briefcase of coins.", choices:[{text:"Take 500 🪙 (+40 Suspicion)",action:()=>{updateCoins(c=>c+500);setSuspicion(s=>Math.min(maxSuspicion,s+40));setView('EXPLORE')}},{text:"Walk Away",action:()=>setView('EXPLORE')}]},
          { title:"Street Thugs", text:"A gang corners you, demanding a toll.", choices:[{text:"Pay up (-100 🪙)",action:()=>{updateCoins(c=>Math.max(0,c-100));setView('EXPLORE')}},{text:"Fight! (Sudden Death)",action:()=>{startArena(false,200,4);}}]},
          { title:"Rest Stop", text:"You find a quiet place away from the Judges.", choices:[{text:"Rest (-30 Suspicion)",action:()=>{setSuspicion(s=>Math.max(0,s-30));setView('EXPLORE')}}]},
          { title:"The Fortune Teller", text:"Cross my palm with silver.", choices:[{text:"Pay 150 🪙",action:()=>{if(coins>=150){updateCoins(c=>c-150);setBlessedFusion(true);setView('EXPLORE')}else showToast("Not enough coins",true)}},{text:"Leave",action:()=>setView('EXPLORE')}]}
        ];
        setActiveEvent(events[Math.floor(Math.random()*events.length)]);
        setView('EVENT');
        return;
      }
      setEncLoc(loc);
      generateEncounter(loc);
    } else if (coins >= loc.cost) {
      updateCoins(c => c - loc.cost);
      setUnlockedLocs(p => [...p, loc.id]);
      showToast(`Unlocked ${loc.name}!`);
    } else {
      showToast(`Need ${loc.cost} coins!`, true);
    }
  }, [unlockedLocs, coins, cityHeat, gameMode, maxSuspicion, updateCoins, showToast, triggerAmbush, generateEncounter]);

  const scanWild = useCallback((cost) => {
    if (coins >= cost) { updateCoins(c=>c-cost); setIsScanned(true); showToast('Data acquired!'); }
    else showToast(`Need ${cost} coins!`, true);
  }, [coins, updateCoins, showToast]);

  const tryCatch = useCallback((move) => {
    let adjustedEnemy = { ...encTarget };
    if (!ambushActive && move === spamTracker.move) {
      const newCount = spamTracker.count + 1;
      setSpamTracker({ move, count: newCount });
      const hasAbsoluteBias = encTarget.rock >= 90 || encTarget.paper >= 90 || encTarget.scissors >= 90;
      if (newCount >= 2 && !hasAbsoluteBias) {
        const counter = getCounterMove(move);
        if (counter==='R') adjustedEnemy.rock += 60;
        if (counter==='P') adjustedEnemy.paper += 60;
        if (counter==='S') adjustedEnemy.scissors += 60;
      }
    } else { setSpamTracker({ move, count: 1 }); }

    const isBoss = adjustedEnemy.rarity === 'Boss';
    const res = evaluateMatch(move, rollMoveFiltered(adjustedEnemy));

    if (res === 1) {
      playSound('vocal_win');
      if (ambushActive) {
        showToast('ESCAPED THE AMBUSH!'); setCityHeat(0); setAmbushActive(false); setCatchRes('WIN');
      } else {
        updateCoins(c => c + (isBoss ? 200 : adjustedEnemy.rarity==='Epic' ? 100 : adjustedEnemy.rarity==='Rare' ? 60 : 30));
        const cur = inventoryMap[adjustedEnemy.id] || { copies:0, level:1, heat:0 };
        markDiscovered(adjustedEnemy.id);
        setCatchRes(cur.level < 10 && cur.copies+1 >= getDupsReq(cur.level) ? 'LEVEL_UP' : 'WIN');
        setInventoryMap(p => {
          const prev = p[adjustedEnemy.id] || { copies:0, level:1, heat:0 };
          if (prev.level >= 10) return { ...p, [adjustedEnemy.id]: prev };
          let [nc, nl] = [prev.copies+1, prev.level];
          if (nc >= getDupsReq(prev.level)) { nc -= getDupsReq(prev.level); nl++; }
          return { ...p, [adjustedEnemy.id]: { ...prev, copies: nc, level: nl } };
        });
        setCityHeat(h => Math.min(100, h + (cityEvent.id==='CRACKDOWN' ? 30 : 15)));
      }
    } else {
      playSound('loser');
      if (ambushActive) {
        const penalty = Math.floor(coinsRef.current * 0.20);
        updateCoins(c => Math.max(0, c-penalty));
        setCityHeat(0); setAmbushActive(false); setCatchRes('PENALTY');
        showToast(`BUSTED! Fined ${penalty} 🪙`, true); triggerShake();
      } else {
        setCatchRes('PENALTY'); updateCoins(c => Math.max(0, c-(isBoss?50:15))); triggerShake();
      }
    }
  }, [encTarget, ambushActive, spamTracker, inventoryMap, cityEvent, playSound, updateCoins, markDiscovered, showToast, triggerShake]);

  // ── TRAINING ───────────────────────────────────────────────────────────────
  const handleTrain = useCallback(() => {
    if (!trainTarget) return;
    const card = getCard(trainTarget);
    if (card?.level >= 10) return showToast('Card is Max Level!', true);
    const cost = card.level * 150;
    if (coins < cost) return showToast(`Need ${cost} coins!`, true);
    updateCoins(c => c - cost);
    setInventoryMap(prev => ({
      ...prev,
      [trainTarget]: { ...prev[trainTarget], level: prev[trainTarget].level + 1, copies: 0 }
    }));
    showToast(`${card.name} Leveled Up to L${card.level+1}!`);
    setTrainTarget(null);
  }, [trainTarget, getCard, coins, updateCoins, showToast]);

  const buyRelic = useCallback((r) => {
    if (coins >= r.cost && !relics.includes(r.id)) {
      updateCoins(c => c - r.cost); setRelics(p => [...p, r.id]); showToast('Relic Acquired.');
    } else showToast('Not enough coins.', true);
  }, [coins, relics, updateCoins, showToast]);

  const buyTrick = useCallback((t) => {
    if (tricks.length >= 3) return showToast('Trick Inventory Full!', true);
    if (coins >= t.cost) { updateCoins(c => c-t.cost); setTricks(p=>[...p,t.id]); showToast(`${t.name} Acquired.`); }
    else showToast('Not enough coins.', true);
  }, [tricks, coins, updateCoins, showToast]);

  const buyDarkWeb = useCallback((bossId, cost) => {
    if (coins < cost) return showToast('Insufficient Funds.', true);
    updateCoins(c => c-cost);
    setInventoryMap(prev => ({ ...prev, [bossId]: { copies:0, level:1, heat:0 } }));
    markDiscovered(bossId);
    showToast('Shipment acquired. Do not speak of this.');
  }, [coins, updateCoins, markDiscovered, showToast]);

  // ── BATTLE ENGINE ──────────────────────────────────────────────────────────
  const generateEnemyTeam = useCallback((isBossRound = false) => {
    if (isBossRound) playSound('boss');
    const pool = [...INITIAL_CARDS, ...customCards];
    if (isBossRound) {
      setEnemyTeam([pool[Math.floor(Math.random()*pool.length)], BOSS_CARDS[Math.floor(Math.random()*BOSS_CARDS.length)], pool[Math.floor(Math.random()*pool.length)]]);
    } else {
      setEnemyTeam([pool[Math.floor(Math.random()*pool.length)], pool[Math.floor(Math.random()*pool.length)], pool[Math.floor(Math.random()*pool.length)]]);
    }
  }, [customCards, playSound]);

  const handleResetToArena = useCallback(() => {
    generateEnemyTeam(false);
    dispatchBattleFull({ type: 'RESET' });
    setCityEvent(CITY_EVENTS[Math.floor(Math.random()*CITY_EVENTS.length)]);
    setView('BATTLE_PREP');
  }, [generateEnemyTeam]);

  useEffect(() => {
    if (view === 'BATTLE_PREP' && enemyTeam.length === 0) generateEnemyTeam();
  }, [view, enemyTeam.length, generateEnemyTeam]);

  const startArena = useCallback((isStreak = false, pot = 0, round = 1) => {
    if (teamIds.slice(0,3).includes(null)) return showToast('Fill 3 Fighters!', true);
    if (!isStreak && coins < wager) return showToast('Not enough coins!', true);
    if (!isStreak) playSound('good_luck');
    const rivalTier = ARENA_RIVALS[Math.min(round-1, ARENA_RIVALS.length-1)];
    const rival = rivalTier[Math.floor(Math.random()*rivalTier.length)];
    let actualPot = pot;
    if (!isStreak) {
      updateCoins(c => c-wager);
      if (!relics.includes('r6')) setSuspicion(rival.suspBuff || 0);
      actualPot = relics.includes('r6') ? wager*4 : wager*2;
    } else { setSuspicion(s => Math.min(maxSuspicion, s+(rival.suspBuff||0))); }
    setCrowdHype(h => Math.min(100, h+10));

    const stage = TOURNAMENT_STAGES[round-1] || TOURNAMENT_STAGES[TOURNAMENT_STAGES.length-1];
    let mod = ARENA_MODIFIERS[0];
    if (stage.modOverride) {
      if (stage.modOverride==='SUDDEN_DEATH') mod = { id:'SUDDEN_DEATH', name:'SUDDEN DEATH', color:'bg-red-900 text-white animate-pulse' };
      else if (stage.modOverride==='BLIND') mod = ARENA_MODIFIERS.find(m=>m.id==='BLIND');
    } else {
      mod = ARENA_MODIFIERS[Math.floor(Math.random()*ARENA_MODIFIERS.length)];
    }

    dispatchBattleFull({ type: 'INIT', payload: {
      phase:'SELECT_FIGHTER', roundNum:0, usedFighters:[], currentPIdx:null,
      pot:actualPot, streak:round, pScore:0, eScore:0,
      matches:[], animKey:Date.now(), modifier:mod, isSuddenDeath:stage.modOverride==='SUDDEN_DEATH', stage,
      clashData:{ pPwrBonus:0, ePwrBonus:0 }, cheatedThisMatch:false,
      synergy:activeSynergy, forcedCheat:null, shuffledMoves:['R','P','S'],
      rival, activeTrick:null, isRigged:false, accused:false, prediction:null,
      activePlayerCard:null, activeEnemyCard:null, activeSupportCard:null, slashAnim:null
    }});
    setView('BATTLE_PLAY');
  }, [teamIds, coins, wager, relics, maxSuspicion, activeSynergy, updateCoins, showToast, playSound]);

  const selectFighter = useCallback((idx) => {
    if (battleState?.usedFighters?.includes(idx)) return;
    let finalEnemyTeam = [...enemyTeam];
    const pCard = getCard(teamIds[idx]);
    const eCard = enemyTeam[battleState.roundNum];

    if (Math.random() < 0.001) {
      finalEnemyTeam[battleState.roundNum] = { ...eCard, rock:0, paper:0, scissors:0, nova:100, name:'GLITCH', emoji:'👾' };
      setEnemyTeam(finalEnemyTeam);
      showToast('⚠️ SCRIPT BREAK: REALITY COMPROMISED', true);
      triggerShake();
    } else {
      const trapChance = battleState.synergy !== 'ASSASSINS' && battleState.synergy !== 'CHIMERAS'
        ? (0.15 + (battleState.rival?.trapMod || 0)) : 0;
      if (Math.random() < trapChance && !battleState.isSuddenDeath) {
        const pool = [...INITIAL_CARDS];
        finalEnemyTeam[battleState.roundNum] = pool[Math.floor(Math.random()*pool.length)];
        setEnemyTeam(finalEnemyTeam);
        showToast('⚠️ RIVAL TRAP! Enemy Swapped Fighters!', true);
        triggerShake();
      }
    }

    let newPot = battleState.pot;
    if (eCard?.id === 'b3') { newPot *= 2; showToast('🦊 Gambler Fox: POT DOUBLED!', true); }
    let shuffledMoves = ['R','P','S'];
    if (eCard?.id === 'b2') { shuffledMoves = [...shuffledMoves].sort(() => Math.random()-0.5); showToast('👁️ ORACLE TORTOISE: UI SCRAMBLED!', true); }
    const isRigged = battleState.synergy !== 'MONOLITH' && eCard?.id !== 'glitch' && battleState.roundNum > 0 && Math.random() < 0.15;

    dispatchBattleFull({ type: '_FN', fn: pr => ({
      ...pr, phase:'PREDICT', currentPIdx:idx, usedFighters:[...pr.usedFighters, idx],
      cheatedThisMatch:false, animKey:Date.now(), bluffActive:false, pot:newPot,
      forcedCheat:null, shuffledMoves, showText:null, isRigged, accused:false, prediction:null,
      activePlayerCard:pCard, activeEnemyCard:finalEnemyTeam[pr.roundNum],
      activeSupportCard:getCard(teamIds[3]), slashAnim:null
    })});
  }, [battleState, enemyTeam, teamIds, getCard, showToast, triggerShake]);

  const handlePrediction = useCallback((m) => {
    dispatchBattleFull({ type: '_FN', fn: pr => {
      let next = { ...pr, prediction:m, phase:'FIGHTING', animKey:Date.now() };
      if (pr.activePlayerCard?.passiveId==='TRICKSTER_SWAP' || pr.activeEnemyCard?.passiveId==='TRICKSTER_SWAP') {
        const tempP = pr.activePlayerCard, tempE = pr.activeEnemyCard;
        next.activePlayerCard = tempE; next.activeEnemyCard = tempP;
        next.showText = { txt:'SWAPPED!', color:'text-fuchsia-400' };
        setTeamIds(prev => { const nt=[...prev]; nt[pr.currentPIdx]=tempE.id; return nt; });
        setEnemyTeam(prev => { const ne=[...prev]; ne[pr.roundNum]=tempP; return ne; });
        setInventoryMap(prev => ({ ...prev, [tempE.id]: { copies:prev[tempE.id]?.copies||0, level:tempE.level||1, heat:tempE.heat||0 } }));
        if ((tempE.id.startsWith('m_')||tempE.id.startsWith('cursed_')) && !customCards.find(c=>c.id===tempE.id)) setCustomCards(prev=>[...prev,tempE]);
        showToast('🃏 TRICKSTER: CARDS SWAPPED PERMANENTLY!'); triggerShake();
      }
      return next;
    }});
  }, [customCards, showToast, triggerShake]);

  const handleAccuse = useCallback(() => {
    if (!battleState || (battleState.phase !== 'FIGHTING' && battleState.phase !== 'REACTION')) return;
    if (battleState.accused) return;
    if (battleState.isRigged) {
      setSuspicion(0);
      showToast('⚖️ CORRUPTION EXPOSED! CROWD UPRISING!');
      triggerShake();
      dispatchBattleFull({ type: '_FN', fn: pr => ({ ...pr, accused:'SUCCESS', phase:'FIGHTING', showText:{ txt:'UPRISING!', color:'text-yellow-400' }, isRigged:false }) });
    } else {
      const newSusp = suspicion + 80;
      setSuspicion(s => Math.min(maxSuspicion, s+80));
      triggerShake();
      dispatchBattleFull({ type: '_FN', fn: pr => {
        const next = { ...pr, accused:'FAILED', showText:{ txt:'CONTEMPT!', color:'text-red-600' } };
        if (newSusp >= maxSuspicion) {
          playSound('illegal');
          next.phase = 'BUSTED';
          if (coinsRef.current <= 0 && !bloodPactOffered) setBloodPactOffered(true);
        }
        return next;
      }});
    }
  }, [battleState, suspicion, maxSuspicion, bloodPactOffered, showToast, triggerShake, playSound]);

  const handleUseTrick = useCallback((trick) => {
    if (battleState?.phase !== 'FIGHTING') return;
    setTricks(prev => prev.filter(t => t !== trick.id));
    dispatchBattleFull({ type: '_FN', fn: pr => ({ ...pr, activeTrick: trick.id }) });
    showToast(`Activated: ${trick.name}!`);
  }, [battleState, showToast]);

  const finishMatchup = useCallback((state, resObj, clashWinner = 0) => {
    let finalRes = clashWinner !== 0 ? clashWinner : resObj.res;
    const pH = resObj.pH, eH = resObj.eH;

    if (pH('CURSE_BLEED') || resObj.sC?.passiveId==='CURSE_BLEED') { updateCoins(c=>Math.max(0,c-10)); showToast('Bleed Curse: -10 🪙', true); }
    if (eH('CURSE_BLEED')) { updateCoins(c=>c+10); showToast('Enemy Bleed: Steal +10 🪙!'); }

    if (state.prediction === resObj.eM) {
      updateCoins(c=>c+10); setCrowdHype(h=>Math.min(100,h+30)); showToast('🎯 PREDICTION CORRECT! +10 🪙');
    } else if (state.prediction !== null && !isRiot) {
      setSuspicion(s=>Math.min(maxSuspicion,s+10));
    }

    if (finalRes === 1) {
      let bonus = 0;
      if (pH('COIN_FARMER') || resObj.sC?.passiveId==='COIN_FARMER') bonus += 15;
      if (state.modifier?.id==='ROCK_FEST' && resObj.pM==='R') bonus += 10;
      if (relics.includes('r3')) { const heal=Math.floor(state.pot*0.05); updateCoins(c=>c+heal); }
      if (state.synergy==='SWARM') bonus += 25;
      if (bonus > 0) updateCoins(c=>c+bonus);
      const baseDecay = state.cheatedThisMatch ? 10 : 20;
      const decay = state.synergy==='KINGS' ? baseDecay+15 : baseDecay;
      setSuspicion(s=>Math.max(0,s-decay));
    } else if (finalRes === -1 && resObj.eC?.id==='b3') {
      updateCoins(c=>Math.max(0,c-50)); showToast('🦊 Gambler Fox stole 50 🪙!', true);
    }

    const pId = teamIds[state.currentPIdx];
    setInventoryMap(prev => {
      if (!prev[pId]) return prev;
      return { ...prev, [pId]: { ...prev[pId], heat: finalRes===1 ? Math.min(5,(prev[pId].heat||0)+1) : 0 } };
    });

    const matchRecord = { pM:resObj.pM, eM:resObj.eM, res:finalRes, wasClash:clashWinner!==0 };
    if (state.isSuddenDeath && finalRes===1 && !state.isUltimateDouble) {
      return { ...state, phase:'ULTIMATE_OFFER', matches:[...state.matches, matchRecord], pScore:state.pScore+1 };
    }
    return { ...state, phase:'REVEAL', matches:[...state.matches, matchRecord], pScore:state.pScore+(finalRes===1?1:0), eScore:state.eScore+(finalRes===-1?1:0), activeTrick:null };
  }, [teamIds, relics, isRiot, maxSuspicion, updateCoins, showToast]);

  const enterReactionPhase = useCallback(() => {
    dispatchBattleFull({ type: '_FN', fn: pr => {
      const pC = pr.activePlayerCard, eC = pr.activeEnemyCard, sC = pr.activeSupportCard;
      let pb = [], eb = [];
      const appB = (pas, isE) => {
        if (!pas) return;
        if (pas==='BLOCK_R') isE ? pb.push('R') : eb.push('R');
        if (pas==='BLOCK_P') isE ? pb.push('P') : eb.push('P');
        if (pas==='BLOCK_S') isE ? pb.push('S') : eb.push('S');
      };
      const isPOverdrive = pC?.heat>=3 || pr.synergy==='CHIMERAS' || pr.accused==='SUCCESS';
      if (!isPOverdrive) appB(eC?.passiveId, true);
      appB(pC?.passiveId, false); appB(sC?.passiveId, false);
      if (pr.modifier?.id==='BAN_S') { pb.push('S'); eb.push('S'); }

      const pH = (id) => pC?.passiveId===id || sC?.passiveId===id;
      const eH = (id) => eC?.passiveId===id;

      let effPC = (eH('SABOTAGE') && !isPOverdrive) || pr.modifier?.id==='CHAOS' ? {...pC,rock:33,paper:33,scissors:34,nova:0} : pC;
      let effEC = pH('SABOTAGE') || pr.modifier?.id==='CHAOS' ? {...eC,rock:33,paper:33,scissors:34,nova:0} : eC;
      if (eH('MIRROR')) effEC = effPC;
      if (pr.isRigged) {
        const dom = getDominantStat(effEC);
        if (dom==='R') effEC.rock+=50; if (dom==='P') effEC.paper+=50; if (dom==='S') effEC.scissors+=50;
      }

      const pM = rollMoveFiltered(effPC, pb, pr.forcedCheat);
      let eM = rollMoveFiltered(effEC, eb);
      if (pr.accused==='SUCCESS') {
        const weak = effEC.rock < effEC.paper ? (effEC.rock < effEC.scissors ? 'R' : 'S') : (effEC.paper < effEC.scissors ? 'P' : 'S');
        eM = weak;
      }

      const rivalIq  = eC.grit + (pr.rival?.iqBuff || 0);
      const reactionMs = pr.activeTrick==='t1' ? 1500 : Math.max(250, Math.min(1200, 500+(pC.iq-rivalIq)*8));
      const readSuccess = pr.activeTrick==='t3' || pr.synergy==='SWARM' || pr.accused==='SUCCESS' || Math.random()*100 < (30+(pC.iq-rivalIq)) || (pr.bluffActive && !pr.rival?.immuneBluff);

      return { ...pr, phase:'REACTION', pendingMoves:{ pM, eM, effPC, pb, effEC, eb, pH, eH }, reactionMs, readSuccess, animKey:Date.now() };
    }});
  }, [showToast]);

  const execResolvePhase = useCallback(() => {
    dispatchBattleFull({ type: '_FN', fn: pr => {
      if (pr.phase !== 'REACTION') return pr;
      const { pM, eM, effPC, pb, effEC, eb, pH, eH } = pr.pendingMoves;
      let finalEM = eM, res = evaluateMatch(pM, finalEM), specialWin = false;

      if (pM === 'N' && finalEM !== 'N') { res = 1; specialWin = true; showToast('NOVA STRIKE! Unblockable Hit!'); triggerShake(); }
      else if (finalEM === 'N' && pM !== 'N') { res = -1; specialWin = true; showToast('Enemy used NOVA STRIKE!', true); triggerShake(); }
      else if (pr.synergy==='ASSASSINS' && finalEM==='P')        { res=1;  specialWin=true; showToast('Assassins execute Paper!'); }
      else if (pH('OMNI_WIN') && Math.random()<0.1)          { res=1;  specialWin=true; showToast('Omni-Win Triggered!'); }
      else if (eH('OMNI_WIN') && Math.random()<0.1 && pr.accused!=='SUCCESS') { res=-1; specialWin=true; showToast('Enemy Omni-Win!',true); triggerShake(); }
      else {
        if (eH('SCISSOR_IMMUNE') && pM==='S')  { res=-1; specialWin=true; showToast('Enemy Immune to Scissors!',true); triggerShake(); }
        if (pH('SCISSOR_IMMUNE') && finalEM==='S') { res=1; specialWin=true; showToast('Immune to Scissors!'); }
        if (res===1 && eH('SECOND_CHANCE') && Math.random()>0.5 && pr.accused!=='SUCCESS') { finalEM=rollMoveFiltered(effEC,eb); res=evaluateMatch(pM,finalEM); specialWin=true; showToast('Enemy Second Chance!',true); }
        if (res===-1 && pH('SECOND_CHANCE') && Math.random()>0.5) { const nP=rollMoveFiltered(effPC,pb); res=evaluateMatch(nP,finalEM); specialWin=true; showToast('Second Chance Reroll!'); }
        if (res===0 && pH('CURSE_FRAGILE'))  { res=-1; specialWin=true; showToast('Fragile Curse: Shattered!',true); triggerShake(); }
        else if (res===0 && eH('CURSE_FRAGILE')) { res=1; specialWin=true; showToast('Enemy Fragile Shattered!'); triggerShake(); }
        if (res===-1 && pH('CURSE_TRAITOR') && Math.random()<0.2) { specialWin=true; showToast('Traitor Curse: Betrayed!',true); triggerShake(); }
        if (res===1 && eH('CURSE_TRAITOR') && Math.random()<0.2)  { specialWin=true; showToast('Enemy Traitor Betrayed them!'); triggerShake(); }
      }

      const resObj = { pM, eM:finalEM, res, specialWin, pC:pr.activePlayerCard, eC:pr.activeEnemyCard, sC:pr.activeSupportCard, pH, eH };

      if (resObj.res===0 && !resObj.specialWin && !(resObj.pH('WIN_TIES')||resObj.eH('WIN_TIES'))) {
        let bonusPwr = (pr.bluffActive?5:0) + (relics.includes('r4')?40:0) + (pr.synergy==='MONOLITH'?30:0) + (pr.activeTrick==='t2'?50:0) + (pr.accused==='SUCCESS'?100:0) + (pr.prediction===finalEM?20:0) + (isRiot?100:0);
        setCrowdHype(h=>Math.min(100,h+20));
        return { ...pr, phase:'CLASH', currentRes:resObj, clashData:{ pPwrBonus:bonusPwr, ePwrBonus:pr.isRigged?50:0 } };
      }
      if (resObj.res===0) {
        if (pH('WIN_TIES') && eH('WIN_TIES')) { return { ...pr, phase:'CLASH', currentRes:resObj, clashData:{ pPwrBonus:0, ePwrBonus:0 } }; }
        else if (pH('WIN_TIES')) resObj.res=1;
        else if (eH('WIN_TIES')) resObj.res=-1;
      }

      if (resObj.res !== 0) triggerShake();
      if (resObj.res === 1) playSound('vocal_win');
      if (resObj.res === -1) playSound('lose');

      const slash = resObj.res===1 ? 'animate-slash-right' : resObj.res===-1 ? 'animate-slash-left' : null;
      return finishMatchup({ ...pr, slashAnim:slash }, resObj);
    }});
  }, [relics, isRiot, playSound, showToast, triggerShake, finishMatchup]);

  const resolveClashPhase = useCallback(() => {
    dispatchBattleFull({ type: '_FN', fn: pr => {
      if (pr.phase !== 'CLASH') return pr;
      const { pC, eC } = pr.currentRes;
      const pTotal = (pC?.pwr||0) + (pr.clashData?.pPwrBonus||0);
      const eTotal = (eC?.pwr||0) + (pr.clashData?.ePwrBonus||0);
      const clashRes = pTotal >= eTotal ? 1 : -1;
      triggerShake();
      if (clashRes===1) playSound('vocal_win'); else playSound('losing');
      const slash = clashRes===1 ? 'animate-slash-right' : 'animate-slash-left';
      return finishMatchup({ ...pr, slashAnim:slash }, pr.currentRes, clashRes);
    }});
  }, [playSound, triggerShake, finishMatchup]);

  useEffect(() => {
    if (crowdHype >= 100 && !isRiot) { setIsRiot(true); showToast('🔥 CROWD RIOT! CHEATING IS FREE! 🔥'); triggerShake(); }
    if (battleState?.phase === 'DONE' && isRiot) { setIsRiot(false); setCrowdHype(0); }
  }, [crowdHype, isRiot, battleState?.phase, showToast, triggerShake]);

  useEffect(() => {
    if (view !== 'BATTLE_PLAY' || !battleState) return;
    const { phase } = battleState;
    
    if (['DONE','BUSTED','SELECT_FIGHTER','PREDICT','ULTIMATE_OFFER'].includes(phase)) {
      setInputLocked(false);
      return;
    }

    let t;
    if (phase === 'FIGHTING') {
      setInputLocked(false);
      t = setTimeout(() => { setInputLocked(false); enterReactionPhase(); }, 1500);
    } else if (phase === 'REACTION') {
      t = setTimeout(() => { setInputLocked(true); execResolvePhase(); }, battleState.reactionMs);
    } else if (phase === 'CLASH') {
      setInputLocked(false);
    } else if (phase === 'REVEAL') {
      setInputLocked(true);
      t = setTimeout(() => {
        setInputLocked(false);
        if (battleState.isSuddenDeath || battleState.isUltimateDouble || battleState.roundNum >= 2) {
          if (battleState.pScore <= battleState.eScore && coinsRef.current <= 0 && !bloodPactOffered) setBloodPactOffered(true);
          dispatchBattleFull({ type: '_FN', fn: pr => ({ ...pr, phase: pr.pScore <= pr.eScore && coinsRef.current <= 0 ? 'BUSTED' : 'DONE' }) });
        } else {
          dispatchBattleFull({ type: '_FN', fn: pr => ({ ...pr, phase:'SELECT_FIGHTER', roundNum:pr.roundNum+1, currentPIdx:null, animKey:Date.now(), slashAnim:null }) });
        }
      }, 2500);
    }
    return () => clearTimeout(t);
  }, [battleState?.phase, battleState?.roundNum, view, enterReactionPhase, execResolvePhase, bloodPactOffered]);

  useEffect(() => {
    if (battleState?.phase !== 'CLASH') return;
    let ticks = 30;
    setClashTime(ticks);
    const interval = setInterval(() => {
      ticks -= 1;
      setClashTime(ticks);
      const mod = battleState.isRigged ? 3 : 4;
      setPulseSync(ticks % mod === 0);
      dispatchBattleFull({ type: '_FN', fn: pr => pr.phase==='CLASH' ? { ...pr, clashData: { ...pr.clashData, ePwrBonus: pr.clashData.ePwrBonus + Math.floor(Math.random()*4)+1 } } : pr });
      if (ticks <= 0) { clearInterval(interval); resolveClashPhase(); }
    }, 100);
    return () => clearInterval(interval);
  }, [battleState?.phase, battleState?.isRigged, resolveClashPhase]);

  const handleQuickSwap = useCallback((m) => {
    if (inputLocked) return;
    dispatchBattleFull({ type: '_FN', fn: pr => pr.phase==='REACTION' ? { ...pr, pendingMoves:{ ...pr.pendingMoves, pM:m }, swapped:true, showText:{ txt:'QUICK SWAP!', color:'text-cyan-400' } } : pr });
  }, [inputLocked]);

  const handleCheat = useCallback((m) => {
    if (battleState?.phase !== 'FIGHTING' || relics.includes('r4') || battleState.activeEnemyCard?.id==='warden') return;
    
    const pC = battleState.activePlayerCard;
    const actualSuspRate = Math.floor(suspRate * (1-(upgrades.shadow*0.05)));
    let addedSuspicion = 0;
    
    if (m === 'N' && pC?.heat >= 5) {
       dispatchBattleFull({ type: '_FN', fn: pr => ({ ...pr, forcedCheat:'N', showText:{ txt:'NOVA STRIKE ACTIVATED', color:'text-indigo-400 animate-pulse' } }) });
       setCrowdHype(h=>Math.min(100,h+20));
       triggerShake();
       return;
    }

    if (battleState.synergy !== 'KINGS' && !isRiot) addedSuspicion = pC?.heat >= 5 ? 70 : actualSuspRate;
    if (battleState.isRigged) addedSuspicion *= 2;
    
    const newSusp = suspicion + addedSuspicion;
    setSuspicion(newSusp);
    setCrowdHype(h=>Math.min(100,h+5));
    
    dispatchBattleFull({ type: '_FN', fn: pr => {
      const next = { ...pr, cheatedThisMatch:true, forcedCheat:m, showText:{ txt:'RIGGED!', color:'text-red-500' } };
      if (newSusp >= maxSuspicion) { playSound('illegal'); next.phase='BUSTED'; triggerShake(); if (coinsRef.current<=0 && !bloodPactOffered) setBloodPactOffered(true); }
      return next;
    }});
  }, [battleState, relics, suspRate, upgrades.shadow, suspicion, maxSuspicion, isRiot, bloodPactOffered, triggerShake, playSound]);

  const handleBluff = useCallback(() => {
    if (coins < 15 || battleState?.phase !== 'FIGHTING') return;
    if (battleState.rival?.immuneBluff || battleState.activeEnemyCard?.id==='warden') { showToast('Enemy is immune to bluffs!', true); return; }
    updateCoins(c=>c-15);
    showToast('🃏 BLUFF ENGAGED! (+5 PWR, Max Read)');
    dispatchBattleFull({ type: '_FN', fn: pr => ({ ...pr, bluffActive:true, showText:{ txt:'BLUFF!', color:'text-fuchsia-400' } }) });
  }, [coins, battleState, updateCoins, showToast]);

  const handleMash = useCallback((e) => {
    e?.preventDefault();
    if (inputLocked) return;
    const baseBonus = pulseSync ? 15 : 5;
    const bonus = battleState?.isRigged ? Math.floor(baseBonus/2) : baseBonus;
    setClashHitMarker({ id:Date.now(), val:bonus });
    playSound('hit');
    if (pulseSync) { triggerShake(); setCrowdHype(h=>Math.min(100,h+2)); }
    dispatchBattleFull({ type: '_FN', fn: pr => pr.phase==='CLASH' ? { ...pr, clashData:{ ...pr.clashData, pPwrBonus:pr.clashData.pPwrBonus+bonus } } : pr });
  }, [inputLocked, pulseSync, battleState?.isRigged, playSound, triggerShake]);

  const startUltimateDouble = useCallback(() => {
    playSound('all_in');
    setEnemyTeam([BOSS_CARDS[Math.floor(Math.random()*BOSS_CARDS.length)]]);
    dispatchBattleFull({ type: '_FN', fn: pr => ({
      ...pr, phase:'SELECT_FIGHTER', roundNum:0, currentPIdx:null, usedFighters:[],
      isUltimateDouble:true, pot:pr.pot*5, matches:[], animKey:Date.now(), shuffledMoves:['R','P','S']
    })});
  }, [playSound]);

  const handleCashOut = useCallback(() => {
    playSound('vocal_win');
    let msg = `Cashed out ${battleState?.pot} 🪙`;
    if ((battleState?.isSuddenDeath || battleState?.isUltimateDouble) && battleState?.pScore > battleState?.eScore) {
      const boss = enemyTeam[0];
      markDiscovered(boss.id);
      setInventoryMap(prev => ({ ...prev, [boss.id]:{ copies:0, level:1, heat:0 } }));
      msg += ` + Captured ${boss.name}!`;
    }
    updateCoins(c => c + (battleState?.pot || 0));
    showToast(msg);
    handleResetToArena();
  }, [battleState, enemyTeam, updateCoins, markDiscovered, showToast, handleResetToArena, playSound]);

  const handleLetItRide = useCallback(() => {
    const nextRound = (battleState?.streak || 1) + 1;
    const nextStage = TOURNAMENT_STAGES[nextRound-1] || TOURNAMENT_STAGES[TOURNAMENT_STAGES.length-1];
    generateEnemyTeam(nextStage.isBoss);
    const newPot = Math.floor((battleState?.pot||0) * (teamGreed?1.5:1) * (nextStage.mult / (battleState?.stage?.mult||1)));
    const nextRivalTier = ARENA_RIVALS[Math.min(nextRound-1, ARENA_RIVALS.length-1)];
    const nextRival = nextRivalTier[Math.floor(Math.random()*nextRivalTier.length)];
    dispatchBattleFull({ type: '_FN', fn: pr => ({ ...pr, streak:nextRound, pot:newPot, stage:nextStage, rival:nextRival, phase:'WAITING' }) });
    setView('BATTLE_PREP');
  }, [battleState, teamGreed, generateEnemyTeam]);

  const renderDeck = useCallback((onCardClick, isLab = false) => (
    <div className="w-full flex-1 overflow-y-auto custom-scrollbar px-2 min-h-0 relative z-10">
      <DeckGrid
        ids={filteredDeckIds}
        inventoryMap={inventoryMap}
        getCard={getCard}
        onCardClick={onCardClick}
        teamIds={teamIds}
        fusionSlots={fusionSlots}
        sellMode={sellMode}
        isLab={isLab}
        deckSort={deckSort}
      />
    </div>
  ), [filteredDeckIds, inventoryMap, getCard, teamIds, fusionSlots, sellMode, deckSort]);

  const needsBottomNav = !['BATTLE_PLAY','CATCH','EVENT','PACK_OPENING'].includes(view);

  // ══════════════════════════════════════════════════════════════════════════
  // RENDER OVERLAYS & MODALS
  // ══════════════════════════════════════════════════════════════════════════
  const renderTutorial = () => {
    if (!showTutorial) return null;
    return (
      <div className="absolute inset-0 bg-slate-950/95 backdrop-blur-xl z-[500] flex flex-col items-center justify-center p-6 text-center animate-slide-in">
         <h2 className="text-4xl text-cyan-400 font-black mb-6 flex items-center gap-3 drop-shadow-[0_0_15px_cyan]"><Info size={40}/> SURVIVAL GUIDE</h2>
         <div className="bg-slate-900 border border-slate-700 p-6 rounded-2xl w-full max-w-sm mb-8 text-left space-y-4 shadow-2xl">
            <div>
              <h3 className="font-black text-emerald-400 flex items-center gap-2"><MapIcon size={16}/> 1. EXPLORE THE CITY</h3>
              <p className="text-slate-300 text-sm mt-1">Spend coins to scan and fight wild cards. Beat them to capture their DNA and level up your existing copies!</p>
            </div>
            <div>
              <h3 className="font-black text-pink-400 flex items-center gap-2"><FlaskConical size={16}/> 2. MUTATE IN THE LAB</h3>
              <p className="text-slate-300 text-sm mt-1">Combine two Level 2+ cards to create powerful Mutants. Beware: Fusion has a risk of backfiring and cursing the card!</p>
            </div>
            <div>
              <h3 className="font-black text-red-400 flex items-center gap-2"><Swords size={16}/> 3. ENTER THE ARENA</h3>
              <p className="text-slate-300 text-sm mt-1">Bet coins in multi-round tournaments. Wait to see the enemy's throw, then tap quickly to swap... or try to cheat before the match begins!</p>
            </div>
            <div className="bg-slate-800 p-3 rounded-xl border border-slate-600 text-xs text-yellow-300 font-bold italic text-center">
              "Judges hate cheaters. If your Suspicion maxes out, you lose everything."
            </div>
         </div>
         <ActionBtn onClick={() => setShowTutorial(false)} className="bg-blue-600 text-white w-full max-w-sm py-4 text-xl shadow-[0_4px_0_#1e3a8a] border-blue-400">GOT IT</ActionBtn>
      </div>
    );
  };


  if (appState === 'MENU') return (
    <div className="min-h-screen bg-slate-950 flex flex-col items-center justify-center p-6 relative overflow-hidden bg-grid-pattern text-white w-full">
      {renderTutorial()}
      <div className="absolute top-[-10%] left-[-10%] w-[120%] h-[120%] bg-[radial-gradient(circle_at_center,rgba(56,189,248,0.15)_0%,transparent_50%)] animate-pulse pointer-events-none" />
      
      <div className="absolute top-0 w-full bg-yellow-400 text-black font-black text-[10px] py-1 overflow-hidden whitespace-nowrap shadow-[0_2px_10px_yellow] pointer-events-none">
         <div className="inline-block tracking-widest" style={{ animation: 'marquee 20s linear infinite' }}>
           ⚠️ BREAKING: THE GRANDMASTER SPOTTED AT ALIEN CRATER... ⚠️ MUTANT LAB RAIDED BY JUDGES, PRICES SURGE... ⚠️ 50% OFF PREMIUM FOIL PACKS IN CARDBOARD DISTRICT... ⚠️
         </div>
      </div>

      <div className="z-10 flex flex-col items-center w-full max-w-sm mt-8">
        <div className="relative w-64 sm:w-80 mb-8">
          <img src="https://i.imgur.com/JD59tZr.png" alt="Jokenpo Brawlers" className="w-full drop-shadow-[0_0_25px_rgba(34,211,238,0.4)] animate-[float_3s_infinite_ease-in-out]" />
        </div>
        <p className="text-slate-400 font-bold tracking-[0.4em] mb-4 mt-2 text-sm text-center">THE UNDERGROUND TCG</p>
        
        <div className="bg-yellow-900/30 border border-yellow-700 text-yellow-500 text-[10px] font-black px-4 py-2 rounded-xl mb-6 flex items-center justify-center text-center w-full shadow-inner">
          <AlertTriangle size={14} className="mr-2 shrink-0"/> PROGRESS RESETS ON REFRESH
        </div>
        
        <div className="bg-slate-900 border-4 border-slate-800 rounded-2xl p-4 w-full mb-8 flex items-center justify-between shadow-2xl relative overflow-hidden cursor-pointer active:scale-95 transition-transform" onClick={handleDevClick}>
          <div className="absolute top-0 left-0 w-2 h-full bg-yellow-400" />
          <span className="font-black text-slate-300 tracking-widest pl-4">PRESTIGE</span>
          <span className="font-black text-3xl text-yellow-400 flex items-center gap-2 drop-shadow-[0_0_10px_orange]">{trophies} <Trophy size={28}/></span>
        </div>

        <div className="space-y-4 w-full flex flex-col items-center">
          <ActionBtn onClick={safeAction(() => startRun('ROGUELITE'))} className="bg-emerald-600 text-white w-full text-2xl py-5 shadow-[0_6px_0_#047857] border-emerald-400">NEW RUN</ActionBtn>
          <div className="flex gap-4 w-full">
            <ActionBtn onClick={safeAction(() => setAppState('SYNDICATE'))} className="flex-1 bg-purple-800 text-white text-lg py-4 shadow-[0_6px_0_#4c1d95] flex justify-center gap-2 border-purple-500"><Award/> SYNDICATE</ActionBtn>
            <ActionBtn onClick={safeAction(() => setAppState('COMPENDIUM'))} className="flex-1 bg-blue-800 text-white text-lg py-4 shadow-[0_6px_0_#1e3a8a] flex justify-center gap-2 border-blue-500"><BookOpen/> ARCHIVE</ActionBtn>
          </div>
          <button onClick={() => setShowTutorial(true)} className="text-cyan-400 font-bold tracking-widest text-xs uppercase mt-4 hover:text-white transition-colors border-b border-transparent hover:border-cyan-400">How to play</button>
        </div>
      </div>
    </div>
  );

  if (appState === 'COMPENDIUM') {
    const allBaseCards = [...INITIAL_CARDS, ...BOSS_CARDS];
    const collectedCount = allBaseCards.filter(c => discovered.includes(c.id)).length;
    return (
      <div className="min-h-screen bg-slate-950 flex flex-col p-6 relative overflow-hidden bg-grid-pattern text-white w-full">
        <button onClick={safeAction(() => setAppState('MENU'))} className="text-slate-400 mb-6 flex items-center gap-2 font-bold uppercase tracking-widest active:scale-95 w-max hover:text-white transition-colors"><ChevronLeft/> Back to Menu</button>
        <h2 className="text-4xl font-black text-cyan-400 drop-shadow-[0_0_15px_rgba(34,211,238,0.8)] tracking-widest uppercase mb-2 flex items-center gap-3"><BookOpen size={36}/> Compendium</h2>
        <div className="bg-slate-900 border-2 border-slate-700 rounded-xl p-4 w-full mb-6 flex items-center justify-between shadow-lg max-w-2xl mx-auto">
          <span className="font-black text-slate-300 tracking-widest">DISCOVERED</span>
          <span className="font-black text-2xl text-cyan-400">{collectedCount} / {allBaseCards.length}</span>
        </div>
        <div className="grid grid-cols-3 sm:grid-cols-5 md:grid-cols-6 gap-3 overflow-y-auto pb-10 flex-1 content-start w-full max-w-5xl mx-auto custom-scrollbar pr-2 z-10">
          {allBaseCards.map(c => (
            <div key={c.id} className={`${discovered.includes(c.id) ? '' : 'brightness-0 opacity-30 saturate-0'} transition-all`}>
              <CardView card={{...c, level:1, pwr:c.pwr}} isLabDisabled={!discovered.includes(c.id)} />
            </div>
          ))}
        </div>
      </div>
    );
  }

  if (appState === 'SYNDICATE') return (
    <div className="min-h-screen bg-slate-950 flex flex-col p-6 relative overflow-hidden bg-grid-pattern text-white w-full">
      <button onClick={safeAction(() => setAppState('MENU'))} className="text-slate-400 mb-6 flex items-center gap-2 font-bold uppercase tracking-widest active:scale-95 w-max hover:text-white transition-colors"><ChevronLeft/> Back to Menu</button>
      <h2 className="text-4xl font-black text-purple-500 drop-shadow-[0_0_15px_rgba(168,85,247,0.8)] tracking-widest uppercase mb-2 flex items-center gap-3"><Award size={36}/> The Syndicate</h2>
      <p className="text-purple-300 font-mono text-xs mb-8 opacity-80">"Spend Trophies for permanent perks."</p>
      <div className="bg-slate-900 border-2 border-slate-700 rounded-xl p-4 w-full mb-8 flex items-center justify-between shadow-lg max-w-md mx-auto">
        <span className="font-black text-slate-300 tracking-widest">TROPHIES</span>
        <span className="font-black text-2xl text-yellow-400 flex items-center gap-2">{trophies} <Trophy size={20}/></span>
      </div>
      <div className="space-y-4 flex-1 overflow-y-auto pb-10 w-full max-w-md mx-auto custom-scrollbar pr-2 z-10">
        {SYNDICATE_UPGRADES.map(u => (
          <div key={u.id} className="bg-slate-900 border-2 border-purple-900 p-4 rounded-xl flex flex-col gap-3 shadow-lg relative overflow-hidden hover:border-purple-500 transition-colors">
            <div className="flex justify-between items-start">
              <div className="pr-4">
                <h3 className="font-black text-lg text-white leading-tight">{u.name}</h3>
                <p className="text-slate-400 text-xs font-bold mt-1 leading-snug">{u.desc}</p>
              </div>
              <div className="bg-purple-950 border border-purple-500 text-purple-300 text-xs font-black px-2 py-1 rounded whitespace-nowrap shrink-0">LVL {upgrades[u.id]}/{u.max}</div>
            </div>
            <div className="flex justify-end mt-2">
              {upgrades[u.id] < u.max ? (
                <ActionBtn onClick={safeAction(() => buyUpgrade(u))} disabled={trophies < u.cost} className="bg-purple-600 text-white text-xs px-6 py-2.5 shadow-[0_4px_0_#4c1d95] w-full">UPGRADE ({u.cost} 🏆)</ActionBtn>
              ) : (
                <div className="text-emerald-500 font-black text-sm border-2 border-emerald-500 px-4 py-2 rounded bg-emerald-950 w-full text-center flex items-center justify-center gap-2"><Star size={16}/> MAX LEVEL ACHIEVED</div>
              )}
            </div>
          </div>
        ))}
      </div>
    </div>
  );

  if (appState === 'SUMMARY' && runSummary) return (
    <div className="min-h-screen bg-slate-950 flex flex-col items-center justify-center p-6 relative overflow-hidden scanlines text-white w-full">
      <div className="z-10 flex flex-col items-center w-full max-w-sm animate-slide-in">
        <BarChartBig size={64} className={`${runSummary.bankrupt ? 'text-red-500' : 'text-emerald-400'} mb-6`} />
        <h1 className={`text-6xl font-black tracking-tighter text-center leading-none mb-2 ${runSummary.bankrupt ? 'text-red-600 drop-shadow-[0_0_15px_red]' : 'text-emerald-500 drop-shadow-[0_0_15px_rgba(16,185,129,0.8)]'}`}>
          {runSummary.bankrupt ? 'BANKRUPT' : 'RETIRED'}
        </h1>
        <p className="text-slate-400 font-bold tracking-[0.2em] mb-12 text-sm uppercase">Run Complete</p>
        <div className="bg-slate-900 border-4 border-slate-800 rounded-2xl p-6 w-full mb-8 flex flex-col gap-4 shadow-2xl">
          {[['FINAL STASH', `${runSummary.coins} 🪙`, 'text-yellow-400'], ['BOSSES CAPTURED', runSummary.bosses, 'text-red-400'], ['ASCENDED CARDS', runSummary.ascended, 'text-cyan-400']].map(([label, val, cls]) => (
            <div key={label} className="flex justify-between items-center border-b border-slate-700 pb-2">
              <span className="font-bold text-slate-400 text-sm">{label}</span>
              <span className={`font-black text-xl ${cls}`}>{val}</span>
            </div>
          ))}
          <div className="mt-4 bg-yellow-900/30 border border-yellow-700 rounded-xl p-4 flex justify-between items-center">
            <span className="font-black text-yellow-500 tracking-widest">TROPHIES EARNED</span>
            <span className="font-black text-3xl text-yellow-400 drop-shadow-[0_0_10px_orange]">+{runSummary.trophies} 🏆</span>
          </div>
        </div>
        <ActionBtn onClick={safeAction(() => setAppState('MENU'))} className="bg-blue-600 text-white w-full text-2xl py-4 shadow-[0_6px_0_#1e3a8a] border-blue-400">RETURN TO MENU</ActionBtn>
      </div>
    </div>
  );

  return (
    <div className={`min-h-screen bg-slate-950 text-slate-100 font-sans select-none overflow-hidden relative flex flex-col font-['Inter',sans-serif] bg-grid-pattern transition-transform duration-75 ${shakeScreen ? 'animate-shake' : ''}`}>
      
      {(showDarkWeb || ambushActive || battleState?.phase === 'BUSTED') && <div className="pointer-events-none absolute inset-0 z-[999] scanlines mix-blend-overlay opacity-50" />}
      
      {toast && (
        <div className={`absolute top-24 left-1/2 -translate-x-1/2 ${toast.isBad ? 'bg-red-600 text-white' : 'bg-yellow-400 text-black'} border-4 border-black px-6 py-2 rounded-full font-black shadow-[4px_4px_0_#000] z-[1000] animate-bounce whitespace-nowrap`}>
          {toast.msg}
        </div>
      )}

      {inspectedCardId && (() => {
        const card = getCard(inspectedCardId);
        if (!card) return null;
        const isBoss = card.rarity === 'Boss';
        const val = (card.rarity==='Epic'?80:card.rarity==='Rare'?40:card.rarity==='Mutant'?100:15) * card.level;
        const trainCost = card.level * 150;
        return (
          <div className="absolute inset-0 bg-slate-950/95 backdrop-blur-xl z-[200] flex flex-col items-center p-6 animate-slide-in overflow-y-auto" onClick={() => setInspectedCardId(null)}>
            <div className="w-full max-w-sm flex flex-col items-center mt-10" onClick={e => e.stopPropagation()}>
              <div className="w-64 mb-8"><CardView card={card} /></div>
              <div className="bg-slate-900 border border-slate-700 rounded-3xl p-6 w-full shadow-2xl">
                <h3 className="font-black text-2xl mb-1 text-center">{card.name}</h3>
                <p className="text-slate-400 text-sm italic text-center mb-6">"{card.flavor}"</p>
                <div className="space-y-3 mb-8">
                  {[['✊','rock','stone'],['✋','paper','emerald'],['✌️','scissors','orange']].map(([icon,key,col]) => (
                    <div key={key} className="flex items-center gap-3">
                      <span className="w-6 text-xl">{icon}</span>
                      <div className="flex-1 h-3 bg-slate-800 rounded-full overflow-hidden"><div className={`h-full bg-${col}-400`} style={{width:`${card[key]}%`}} /></div>
                      <span className="text-xs font-black w-8 text-right">{Math.round(card[key])}</span>
                    </div>
                  ))}
                  {[['iq','Brain','blue',Brain],['grit','Glasses','purple',Glasses]].map(([key,label,col,Icon]) => (
                    <div key={key} className="flex items-center gap-3 pt-2 border-t border-slate-800">
                      <span className="w-6 text-slate-500"><Icon size={18}/></span>
                      <div className="flex-1 h-2 bg-slate-800 rounded-full overflow-hidden"><div className={`h-full bg-${col}-400`} style={{width:`${Math.min(100,card[key])}%`}} /></div>
                      <span className={`text-xs font-black w-8 text-right text-${col}-300`}>{card[key]}</span>
                    </div>
                  ))}
                </div>
                <div className="grid grid-cols-2 gap-3 mb-4">
                  {[0,1,2].map(i => <ActionBtn key={i} onClick={safeAction(()=>{equipFromInspect(i);setInspectedCardId(null);})} className="bg-blue-900 border-blue-600 text-white text-xs py-3 shadow-[0_4px_0_#1e3a8a]">Equip F{i+1}</ActionBtn>)}
                  <ActionBtn onClick={safeAction(()=>{equipFromInspect(3);setInspectedCardId(null);})} className="bg-amber-700 border-amber-500 text-white text-xs py-3 shadow-[0_4px_0_#78350f]">Equip Support</ActionBtn>
                </div>
                <div className="flex gap-3 mb-4">
                  {card.level < 10 ? (
                    <ActionBtn onClick={safeAction(()=>{if(coins<trainCost)return showToast(`Need ${trainCost} coins!`,true);updateCoins(c=>c-trainCost);setInventoryMap(prev=>({...prev,[card.id]:{...prev[card.id],level:prev[card.id].level+1,copies:0}}));showToast('Leveled Up!');})} className="flex-1 bg-amber-500 text-black text-xs py-3 shadow-[0_4px_0_#b45309] border-amber-300">Train (-{trainCost}🪙)</ActionBtn>
                  ) : (
                    <div className="flex-1 bg-cyan-900 border-cyan-500 text-cyan-300 text-xs py-3 rounded-xl flex items-center justify-center font-black">MAX LEVEL</div>
                  )}
                  <ActionBtn onClick={safeAction(()=>{if(teamIds.includes(card.id))return showToast('Unequip first!',true);if(isBoss)return showToast('Cannot scrap Bosses!',true);updateCoins(c=>c+val);setFusionSlots(prev=>[prev[0]===card.id?null:prev[0],prev[1]===card.id?null:prev[1]]);setInventoryMap(prev=>{const next={...prev};delete next[card.id];return next;});if(card.id.startsWith('m_')||card.id.startsWith('cursed_'))setCustomCards(prev=>prev.filter(c=>c.id!==card.id));showToast(`Scrapped for ${val} 🪙`);setInspectedCardId(null);})} disabled={teamIds.includes(card.id)||isBoss} className="flex-1 bg-red-800 text-white text-xs py-3 shadow-[0_4px_0_#7f1d1d] border-red-500">Scrap (+{val}🪙)</ActionBtn>
                </div>
                <ActionBtn onClick={() => setInspectedCardId(null)} className="w-full bg-slate-800 text-slate-300 border-slate-600 py-4 shadow-[0_4px_0_#334155]">CLOSE INSPECT</ActionBtn>
              </div>
            </div>
          </div>
        );
      })()}

      {showDarkWeb && (
        <div className="absolute inset-0 bg-black/95 z-[200] flex flex-col items-center p-6 animate-slide-in overflow-y-auto">
          <button onClick={() => setShowDarkWeb(false)} className="absolute top-6 right-6 text-red-500 hover:text-white bg-red-950/50 p-2 rounded-full border border-red-900"><Lock size={24} className="rotate-45"/></button>
          <div className="text-center mt-4 mb-8">
            <h2 className="text-4xl font-black text-red-600 drop-shadow-[0_0_15px_red] tracking-widest uppercase flex items-center justify-center gap-3"><Skull size={32}/> THE DARK WEB</h2>
            <p className="text-red-400 font-mono text-xs mt-2 bg-red-950/50 inline-block px-3 py-1 rounded-full border border-red-900">"Everything has a price."</p>
          </div>
          <div className="w-full max-w-sm space-y-4">
            <div className="bg-slate-900 border border-slate-700 p-4 rounded-2xl shadow-xl flex flex-col gap-4">
              <div className="flex items-center gap-4">
                <div className="text-4xl bg-slate-950 p-3 rounded-xl border border-slate-800 shadow-inner">🚨</div>
                <div><h3 className="font-black text-lg text-white leading-tight">Wipe Record</h3><p className="text-slate-400 text-xs font-bold leading-tight">Instantly resets City Heat to 0%.</p></div>
              </div>
              <ActionBtn onClick={safeAction(()=>{if(coins<5000)return showToast('Insufficient Funds.',true);updateCoins(c=>c-5000);setCityHeat(0);showToast('Record Wiped.');setShowDarkWeb(false);})} className="w-full bg-slate-800 text-white text-xs py-3 shadow-[0_4px_0_#0f172a] border-slate-600">PAY 5,000 🪙</ActionBtn>
            </div>
            {BOSS_CARDS.filter(b => b.id !== 'glitch').map(boss => (
              <div key={boss.id} className="bg-slate-900 border border-slate-700 p-4 rounded-2xl shadow-xl flex flex-col gap-4">
                <div className="flex items-center gap-4">
                  <div className="text-4xl bg-slate-950 p-3 rounded-xl border border-slate-800 shadow-inner">{boss.emoji}</div>
                  <div><h3 className="font-black text-lg text-red-400 leading-tight">{boss.name}</h3><p className="text-slate-400 text-xs font-bold leading-tight">Bounty Contract.</p></div>
                </div>
                <ActionBtn onClick={safeAction(()=>{if(coins<25000)return showToast('Insufficient Funds.',true);buyDarkWeb(boss.id,25000);setShowDarkWeb(false);})} className="w-full bg-red-900 text-red-200 text-xs py-3 shadow-[0_4px_0_#450a0a] border-red-700 hover:bg-red-800">HIRE FOR 25,000 🪙</ActionBtn>
              </div>
            ))}
          </div>
        </div>
      )}

      {showRetireConfirm && (
        <div className="absolute inset-0 bg-black/95 z-[100] flex flex-col items-center justify-center p-6 animate-slide-in">
          <LogOut size={64} className="text-red-500 mb-4 animate-pulse" />
          <h2 className="text-4xl font-black text-white mb-2 tracking-widest text-center">RETIRE RUN?</h2>
          <p className="text-slate-400 text-center mb-8 text-sm font-bold max-w-xs">Cash in your team and coins for Trophies, ending this run permanently.</p>
          <ActionBtn onClick={safeAction(() => endRun(false))} className="bg-emerald-600 text-white w-full max-w-xs text-2xl py-4 shadow-[0_6px_0_#047857] mb-4">CASH IN RUN</ActionBtn>
          <button onClick={() => setShowRetireConfirm(false)} className="text-slate-500 font-black uppercase tracking-widest py-4 hover:text-white">Cancel</button>
        </div>
      )}

      {bloodPactOffered && (
        <div className="absolute inset-0 bg-red-950/95 z-[150] flex flex-col items-center justify-center p-6 animate-slide-in">
          <div className="absolute inset-0 bg-[radial-gradient(circle_at_center,rgba(220,38,38,0.2)_0%,transparent_70%)] animate-pulse pointer-events-none" />
          <Siren size={80} className="text-red-500 mb-6 animate-bounce drop-shadow-[0_0_20px_red]" />
          <h2 className="text-5xl font-black text-red-500 mb-2 tracking-widest text-center italic drop-shadow-md">BANKRUPT.</h2>
          <p className="text-red-200 text-center mb-8 text-sm font-bold max-w-xs bg-red-900/50 p-4 rounded-xl border border-red-500/50 shadow-inner">The Syndicate offers a bailout:<br/><br/><span className="text-yellow-400 text-lg">1,500 🪙</span><br/><br/>But 3 cards will be permanently cursed.</p>
          <ActionBtn onClick={safeAction(acceptBloodPact)} className="bg-red-700 text-white w-full max-w-xs text-2xl py-6 shadow-[0_8px_0_#450a0a] border-red-400 mb-6 ring-4 ring-red-900">SIGN BLOOD PACT</ActionBtn>
          <button onClick={safeAction(() => endRun(true))} className="text-slate-400 font-black uppercase tracking-widest py-4 hover:text-white border-b border-transparent hover:border-white">Refuse & Die</button>
        </div>
      )}

      <main className={`flex-1 overflow-hidden relative z-10 flex flex-col ${needsBottomNav ? 'pb-24' : 'pb-4'} ${isRiot ? 'animate-riot' : ''}`}>
        
        {view === 'EXPLORE' && (
          <div className="flex flex-col h-full p-4 relative">
            <h2 className="text-3xl font-black text-white drop-shadow-[2px_2px_0_#000] text-center mt-2 tracking-widest uppercase">The City</h2>
            <div className={`w-full max-w-md mx-auto mt-4 mb-2 bg-slate-900/80 backdrop-blur-sm rounded-xl p-3 border border-white/10 shadow-lg text-center ${cityEvent.id !== 'NORMAL' ? 'animate-pulse' : ''}`}>
              <span className={`text-xs font-black tracking-widest uppercase ${cityEvent.color}`}>{cityEvent.name}</span>
              <p className="text-[10px] text-slate-300 font-bold mt-1">{cityEvent.desc}</p>
            </div>
            <div className="w-full max-w-md mx-auto my-2 bg-slate-900/80 backdrop-blur-sm rounded-xl p-3 border border-white/10 shadow-lg">
              <div className="flex justify-between items-center mb-1">
                <span className="text-[10px] font-black text-red-400 tracking-widest flex items-center gap-1"><Siren size={12}/> WANTED LEVEL</span>
                <span className="text-[10px] font-black text-white">{cityHeat}%</span>
              </div>
              <div className="w-full h-2 bg-black rounded-full overflow-hidden relative">
                <div className={`h-full transition-all duration-300 ${cityHeat >= 80 ? 'bg-red-500 animate-pulse' : 'bg-gradient-to-r from-orange-400 to-orange-600'}`} style={{width:`${cityHeat}%`}} />
              </div>
            </div>
            <div className="flex-1 overflow-y-auto pb-10 flex flex-col gap-4 max-w-md mx-auto w-full custom-scrollbar pr-2 z-10">
              {LOCATIONS.map(l => (
                <div key={l.id} onClick={safeAction(() => clickLoc(l))} className={`bg-slate-800/90 backdrop-blur-md border border-white/10 p-4 rounded-2xl shadow-xl flex items-center gap-4 transition-transform ${!unlockedLocs.includes(l.id)&&!['SHOP','GYM','GACHA'].includes(l.id) ? 'brightness-50 grayscale opacity-80' : 'active:translate-y-1 active:shadow-none cursor-pointer hover:bg-slate-700/90 hover:border-white/20'}`}>
                  <div className="text-5xl drop-shadow-md">{l.icon}</div>
                  <div className="flex-1">
                    <h3 className={`font-black text-xl flex items-center gap-2 ${l.id==='SHOP'?'text-fuchsia-400':l.id==='GACHA'?'text-pink-400':l.id==='GYM'?'text-amber-400':'text-white'}`}>{l.name} {!unlockedLocs.includes(l.id) && !['SHOP','GYM','GACHA'].includes(l.id) && <Lock size={18}/>}</h3>
                    <p className={`text-sm font-bold leading-tight ${l.id==='SHOP'?'text-fuchsia-300':l.id==='GACHA'?'text-pink-300':l.id==='GYM'?'text-amber-300':l.difficulty===1?'text-emerald-400':l.difficulty===2?'text-amber-400':'text-red-400'}`}>{l.desc}</p>
                  </div>
                  {unlockedLocs.includes(l.id) && l.cost > 0 && l.difficulty > 0 && <div className="bg-slate-950 border border-slate-700 text-slate-300 text-xs font-black px-2 py-1 rounded-lg">-{l.cost} 🪙</div>}
                  {!unlockedLocs.includes(l.id) && !['SHOP','GYM','GACHA'].includes(l.id) && <ActionBtn className="bg-yellow-500 text-black text-sm px-3 shadow-[0_4px_0_#ca8a04]">Unlock<br/>{l.cost}</ActionBtn>}
                </div>
              ))}
            </div>
          </div>
        )}

        {view === 'GACHA_SHOP' && (
          <div className="flex flex-col h-full p-4 items-center scanlines relative">
            <div className="text-center mt-4 mb-8">
              <h2 className="text-4xl font-black text-pink-500 drop-shadow-[0_0_15px_rgba(236,72,153,0.8)] tracking-widest uppercase animate-pulse flex items-center justify-center gap-2"><PackageOpen size={36}/> Card Shop</h2>
              <p className="text-pink-300 font-bold text-xs mt-2 opacity-80">"Tear open packs for rare fighters!"</p>
            </div>
            <div className="w-full max-w-md space-y-6 flex-1 z-10">
              {[{ type:'BASIC', emoji:'📦', title:'Basic Pack', desc:'3 Random Cards. Low Rare chance.', cost: cityEvent.id==='SALE'?250:500, style:'bg-blue-600 text-white text-xl w-full shadow-[0_6px_0_#1e3a8a] border-blue-400 py-4' },
                { type:'PREMIUM', emoji:'✨🎁✨', title:'Premium Foil Pack', desc:'3 Cards. Guaranteed 1 Epic!', cost: cityEvent.id==='SALE'?750:1500, style:'bg-pink-600 text-white text-xl w-full shadow-[0_6px_0_#9d174d] border-pink-400 py-4 z-10' }
              ].map(({ type, emoji, title, desc, cost, style }) => (
                <div key={type} className={`bg-slate-900/80 backdrop-blur-md border ${type==='PREMIUM'?'border-pink-500/50 shadow-[0_0_30px_rgba(236,72,153,0.3)]':'border-slate-700'} p-6 rounded-3xl flex flex-col items-center gap-2 shadow-2xl text-center relative ${type==='PREMIUM'?'overflow-hidden':''}`}>
                  {cityEvent.id==='SALE' && <div className="absolute -top-3 right-4 bg-fuchsia-600 text-white font-black text-xs px-3 py-1 rounded-full border-2 border-white shadow-[0_0_15px_fuchsia] rotate-12 z-20">-50% OFF</div>}
                  {type==='PREMIUM' && <div className="absolute inset-0 bg-[radial-gradient(circle_at_center,rgba(236,72,153,0.2)_0%,transparent_70%)] animate-pulse z-0" />}
                  <div className="text-6xl drop-shadow-lg mb-2 z-10">{emoji}</div>
                  <h3 className={`font-black text-2xl z-10 ${type==='PREMIUM'?'text-pink-400':'text-white'}`}>{title}</h3>
                  <p className={`text-sm font-bold mb-4 z-10 ${type==='PREMIUM'?'text-pink-200':'text-slate-400'}`}>{desc}</p>
                  <ActionBtn onClick={safeAction(()=>buyPack(type))} className={style}>BUY ({cost} 🪙)</ActionBtn>
                </div>
              ))}
            </div>
          </div>
        )}

        {view === 'PACK_OPENING' && (
          <div className="flex flex-col items-center justify-center h-full p-4 bg-slate-950 scanlines absolute inset-0 z-50">
            {gachaFlipped.every(Boolean) && (
               <div className="absolute inset-0 pointer-events-none overflow-hidden z-[60]">
                 {[...Array(20)].map((_, i) => (
                   <div key={i} className="absolute text-3xl animate-[confetti_3s_ease-in-out_forwards]" style={{ left:`${Math.random()*100}%`, animationDelay:`${Math.random()}s` }}>
                     {['✨','🌟','🎉','🎊','🔥'][Math.floor(Math.random()*5)]}
                   </div>
                 ))}
               </div>
            )}
            <div className="absolute inset-0 bg-[radial-gradient(circle_at_center,rgba(236,72,153,0.15)_0%,transparent_60%)] animate-pulse pointer-events-none" />
            <h2 className="text-4xl font-black text-white mb-12 animate-pulse tracking-widest drop-shadow-[0_0_15px_white] z-10">{packTorn ? 'TAP TO REVEAL' : 'TAP TO TEAR OPEN'}</h2>
            {!packTorn ? (
              <div className="cursor-pointer hover:scale-110 transition-transform active:scale-95 z-10" onClick={() => { setPackTorn(true); triggerShake(); playSound('hit'); }}>
                <div className="text-[150px] drop-shadow-[0_0_40px_rgba(236,72,153,0.8)] animate-bounce">🎁</div>
              </div>
            ) : (
              <div className="flex gap-4 items-center justify-center flex-wrap w-full max-w-lg z-10 perspective-1000">
                {gachaCards.map((c, i) => (
                  <div key={i} className="w-[100px] sm:w-[140px]" onClick={() => { const nf=[...gachaFlipped]; nf[i]=true; setGachaFlipped(nf); playSound('blip'); }}>
                    {gachaFlipped[i] ? <div className="animate-flip"><CardView card={{...c,level:1,copies:0}} /></div> : <CardBack />}
                  </div>
                ))}
              </div>
            )}
            {gachaFlipped.every(Boolean) && (
              <div className="mt-16 animate-slide-in z-10">
                <ActionBtn onClick={safeAction(claimGacha)} className="bg-emerald-500 text-white text-2xl px-12 py-4 shadow-[0_6px_0_#047857] border-emerald-300">Claim Cards!</ActionBtn>
              </div>
            )}
          </div>
        )}

        {view === 'GYM' && (
          <div className="flex flex-col h-full p-4 items-center relative">
            <div className="text-center mt-2 mb-6">
              <h2 className="text-4xl font-black text-amber-500 drop-shadow-[0_0_15px_rgba(245,158,11,0.8)] tracking-widest uppercase animate-pulse flex items-center justify-center gap-2"><Dumbbell/> Iron Dojo</h2>
              <p className="text-amber-300 font-bold text-xs mt-2 bg-slate-900/50 inline-block px-3 py-1 rounded-full">"Pay the fee, break the limits."</p>
            </div>
            {trainTarget ? (
              <div className="w-full max-w-xs flex flex-col items-center bg-slate-800/90 backdrop-blur-md p-6 rounded-3xl border border-white/10 shadow-2xl animate-slide-in z-10">
                <div className="w-48 mb-4"><CardView card={getCard(trainTarget)} /></div>
                {getCard(trainTarget)?.level < 10 ? (
                  <>
                    <p className="text-white font-black text-lg mt-2 mb-4 text-center">Upgrade to Level {getCard(trainTarget).level+1}?</p>
                    <ActionBtn onClick={safeAction(handleTrain)} className="bg-amber-500 text-black w-full text-xl shadow-[0_6px_0_#92400e] border-amber-300 py-4">TRAIN (-{getCard(trainTarget).level*150} 🪙)</ActionBtn>
                  </>
                ) : (
                  <p className="text-cyan-400 font-black text-3xl mt-4 mb-2 animate-pulse drop-shadow-[0_0_15px_cyan]">ASCENDED</p>
                )}
                <button onClick={safeAction(() => setTrainTarget(null))} className="text-slate-400 font-bold uppercase tracking-widest mt-6 hover:text-white transition-colors">Cancel</button>
              </div>
            ) : (
              <div className="w-full max-w-xl bg-slate-800/80 backdrop-blur-md rounded-2xl p-4 border border-white/10 flex-1 overflow-hidden flex flex-col shadow-xl z-10">
                <p className="text-center text-xs font-bold text-amber-400 mb-2 uppercase tracking-widest">Select Card to Train</p>
                {renderDeck(setTrainTarget)}
              </div>
            )}
          </div>
        )}

        {view === 'EVENT' && activeEvent && (
          <div className="flex flex-col items-center justify-center h-full p-6 scanlines relative">
            <div className="bg-slate-900/95 backdrop-blur-xl border border-fuchsia-500/50 p-8 rounded-3xl text-center w-full max-w-sm shadow-[0_0_40px_rgba(192,38,211,0.3)] animate-slide-in relative overflow-hidden">
              <div className="absolute top-0 left-0 w-full h-1 bg-fuchsia-500" />
              <Ghost size={64} className="mx-auto text-fuchsia-400 mb-6 animate-pulse drop-shadow-[0_0_15px_fuchsia]" />
              <h2 className="text-3xl font-black text-white mb-4 uppercase tracking-widest leading-tight">{activeEvent.title}</h2>
              <p className="text-slate-300 font-bold mb-8 italic text-sm">"{activeEvent.text}"</p>
              <div className="flex flex-col gap-4">
                {activeEvent.choices.map((c, i) => (
                  <ActionBtn key={i} onClick={safeAction(c.action)} className={`${i===0?'bg-fuchsia-600 text-white shadow-[0_6px_0_#701a75] border-fuchsia-400':'bg-slate-800 text-slate-300 shadow-[0_4px_0_#0f172a] border-slate-600'} py-4 text-sm`}>{c.text}</ActionBtn>
                ))}
              </div>
            </div>
          </div>
        )}

        {view === 'BLACK_MARKET' && (
          <div className="flex flex-col h-full p-4 items-center scanlines relative">
            <div className="text-center mt-4 mb-6">
              <h2 className="text-4xl font-black text-fuchsia-500 drop-shadow-[0_0_15px_rgba(192,38,211,0.8)] tracking-widest uppercase animate-pulse">Black Market</h2>
              <p className="text-fuchsia-300 font-mono text-xs mt-2 opacity-80 bg-slate-900/50 inline-block px-3 py-1 rounded-full">"Permanent rule breakers & dirty tricks."</p>
            </div>
            <div className="w-full max-w-md flex gap-2 mb-3">
              <div className="flex-1 bg-fuchsia-900/80 backdrop-blur-sm text-white font-black text-xs py-2 rounded-xl text-center border border-fuchsia-500/30">RELICS (PASSIVE)</div>
            </div>
            <div className="w-full max-w-md space-y-3 flex-1 overflow-y-auto pb-4 border-b-2 border-white/5 mb-4 custom-scrollbar pr-2 z-10">
              {RELICS_DB.map(r => (
                <div key={r.id} className="bg-slate-800/90 backdrop-blur-md border border-white/10 p-3 rounded-2xl flex items-center gap-4 shadow-xl relative overflow-hidden">
                  <div className="absolute top-0 left-0 w-1 h-full bg-fuchsia-500" />
                  <div className="text-3xl bg-slate-950/50 p-3 rounded-xl border border-white/5 shadow-inner">{r.icon}</div>
                  <div className="flex-1"><h3 className="font-black text-sm text-white leading-tight">{r.name}</h3><p className="text-slate-400 text-[10px] font-bold mt-1 leading-tight">{r.desc}</p></div>
                  {relics.includes(r.id) ? <div className="text-emerald-400 font-black text-[10px] rotate-[-10deg] border-2 border-emerald-500 px-2 py-1 rounded bg-emerald-950/50">SOLD OUT</div>
                    : <ActionBtn onClick={safeAction(()=>buyRelic(r))} disabled={coins<r.cost} className="bg-fuchsia-700 text-white text-[10px] px-3 py-2 shadow-[0_4px_0_#4a044e] border-fuchsia-500">BUY<br/>{r.cost}</ActionBtn>}
                </div>
              ))}
            </div>
            <div className="w-full max-w-md flex gap-2 mb-3 z-10">
              <div className="flex-1 bg-cyan-900/80 backdrop-blur-sm text-white font-black text-xs py-2 rounded-xl text-center border border-cyan-500/30">TRICKS (CONSUMABLE)</div>
            </div>
            <div className="w-full max-w-md space-y-3 flex-1 overflow-y-auto pb-6 custom-scrollbar pr-2 z-10">
              {TRICKS_DB.map(t => (
                <div key={t.id} className="bg-slate-800/90 backdrop-blur-md border border-white/10 p-3 rounded-2xl flex items-center gap-4 shadow-xl relative overflow-hidden">
                  <div className="absolute top-0 left-0 w-1 h-full bg-cyan-500" />
                  <div className="text-cyan-400 bg-slate-950/50 p-3 rounded-xl border border-white/5 shadow-inner text-2xl">{t.icon}</div>
                  <div className="flex-1"><h3 className="font-black text-sm text-white leading-tight">{t.name}</h3><p className="text-slate-400 text-[10px] font-bold mt-1 leading-tight">{t.desc}</p></div>
                  <ActionBtn onClick={safeAction(()=>buyTrick(t))} disabled={coins<t.cost||tricks.length>=3} className="bg-cyan-600 text-white text-[10px] px-3 py-2 shadow-[0_4px_0_#0e7490] border-cyan-400">BUY<br/>{t.cost}</ActionBtn>
                </div>
              ))}
            </div>
            <div className="w-full max-w-md mt-4 border-t-2 border-red-900 pt-4 z-10">
              <ActionBtn onClick={safeAction(()=>setShowDarkWeb(true))} className="bg-black text-red-600 border-red-900 w-full py-4 tracking-widest animate-pulse"><Skull size={20} className="mr-2"/> ENTER DARK WEB</ActionBtn>
            </div>
          </div>
        )}

        {view === 'CATCH' && encTarget && (
          <div className="flex flex-col items-center justify-center h-full p-4 space-y-6 relative">
            <div className={`bg-slate-900/90 backdrop-blur-xl border border-white/10 p-8 rounded-3xl text-center w-full max-w-sm relative shadow-2xl z-10 ${ambushActive ? 'border-red-500 shadow-[0_0_50px_rgba(220,38,38,0.5)] animate-pulse' : encTarget.rarity==='Boss' ? 'border-red-600 shadow-[0_0_40px_rgba(220,38,38,0.3)]' : ''}`}>
              <h2 className={`text-2xl font-black mb-4 animate-pulse tracking-widest ${encTarget.rarity==='Boss' ? 'text-red-500 drop-shadow-[0_0_10px_red]' : 'text-cyan-400 drop-shadow-[0_0_10px_cyan]'}`}>{ambushActive ? '🚨 POLICE AMBUSH! 🚨' : encTarget.rarity==='Boss' ? '⚠️ BOSS AMBUSH!' : 'WILD ENCOUNTER!'}</h2>
              <div className={`text-[120px] mb-6 drop-shadow-[0_10px_20px_rgba(0,0,0,0.8)] ${catchRes==='PENALTY' ? 'animate-samba' : catchRes ? 'animate-shrink' : ambushActive ? 'animate-[bounce_0.5s_infinite]' : 'animate-[pulse_1s_infinite]'}`}>{encTarget.emoji}</div>
              <h3 className="font-black text-3xl tracking-tight text-white bg-black/50 inline-block px-4 py-1 rounded-full">{encTarget.name}</h3>
              {!catchRes && (
                <div className="mt-8 bg-slate-950/80 p-4 rounded-2xl border border-white/10 relative shadow-inner">
                  <div className="absolute -top-4 -right-2 bg-red-600 text-white text-[10px] font-black px-3 py-1.5 rounded-full shadow-lg border border-red-300 rotate-12">{ambushActive ? 'RISK: 20% OF STASH' : `RISK: ${encTarget.rarity==='Boss'?50:15} 🪙`}</div>
                  {!isScanned && encLoc?.difficulty > 1 && !ambushActive ? (
                    <div className="flex flex-col items-center py-2">
                      <p className="text-xs font-black text-red-400 mb-3 tracking-widest uppercase">HIDDEN BIASES</p>
                      <ActionBtn onClick={safeAction(()=>scanWild((encLoc?.difficulty||1)*20))} className="bg-blue-600 text-white text-sm py-3 px-6 shadow-[0_4px_0_#1e3a8a] border-blue-400 active:shadow-none flex items-center gap-2 w-full"><Scan size={16}/> Scan Data ({(encLoc?.difficulty||1)*20} 🪙)</ActionBtn>
                    </div>
                  ) : (
                    <>
                      <p className="text-[10px] font-black text-emerald-400 mb-3 flex items-center justify-center gap-1 tracking-widest uppercase bg-emerald-950/50 py-1 rounded-lg"><Scan size={12}/> DATA ACQUIRED</p>
                      <div className="flex justify-center gap-8 text-xl font-black text-white">
                        <span className="text-stone-300 drop-shadow-md">✊ {Math.round(encTarget.rock)}%</span>
                        <span className="text-emerald-400 drop-shadow-md">✋ {Math.round(encTarget.paper)}%</span>
                        <span className="text-orange-400 drop-shadow-md">✌️ {Math.round(encTarget.scissors)}%</span>
                      </div>
                    </>
                  )}
                </div>
              )}
            </div>
            {!catchRes ? (
              <div className="w-full max-w-sm flex gap-3 z-10">
                {['R','P','S'].map(m => <ActionBtn key={m} onClick={safeAction(()=>tryCatch(m))} className="flex-1 bg-yellow-500 text-black text-6xl pb-6 pt-4 shadow-[0_6px_0_#a16207] border-yellow-200 active:shadow-none hover:bg-yellow-400">{MOVES[m]}</ActionBtn>)}
              </div>
            ) : (
              <div className="text-center space-y-6 w-full max-w-sm flex flex-col items-center z-10">
                <h2 className={`text-6xl font-black drop-shadow-[0_5px_10px_rgba(0,0,0,0.8)] ${catchRes==='PENALTY'?'text-red-500':'text-emerald-400'} animate-[bounce_0.5s_infinite]`}>{catchRes==='PENALTY'?'FLOPPED!':catchRes==='LEVEL_UP'?'LEVEL UP!':'CAUGHT!'}</h2>
                {catchRes==='PENALTY' && <div className="text-[100px] mt-4 animate-flop drop-shadow-2xl">😵</div>}
                <div className="w-full flex flex-col gap-4 mt-8">
                  <ActionBtn onClick={safeAction(()=>generateEncounter(encLoc))} className="bg-yellow-500 text-black text-2xl shadow-[0_6px_0_#a16207] border-yellow-200 active:shadow-none py-5">Keep Exploring</ActionBtn>
                  <ActionBtn onClick={safeAction(()=>handleNav('EXPLORE'))} className="bg-slate-700 text-white text-xl shadow-[0_4px_0_#334155] border-slate-500 active:shadow-none py-4">Flee to Map</ActionBtn>
                </div>
              </div>
            )}
          </div>
        )}

        {view === 'DECK' && (
          <div className="h-full flex flex-col p-4 items-center relative z-10">
            <div className="w-full max-w-3xl flex justify-between items-center mb-4 bg-slate-900/80 backdrop-blur-md p-3 rounded-2xl border border-white/10 shadow-lg relative">
              <button onClick={safeAction(()=>setShowRetireConfirm(true))} className="p-2.5 bg-red-950/80 border border-red-500/50 text-red-400 rounded-xl hover:bg-red-900 transition-colors shadow-inner active:scale-95"><LogOut size={18}/></button>
              <div className="flex items-center gap-2 text-white font-black text-xl tracking-widest uppercase"><Library size={24} className="text-cyan-400"/> ROSTER</div>
              <button onClick={safeAction(()=>setSellMode(!sellMode))} className={`px-4 py-2 text-xs font-black rounded-xl flex items-center gap-2 transition-colors border ${sellMode?'bg-red-600 border-red-400 text-white animate-pulse shadow-[0_0_15px_red]':'bg-slate-800 border-slate-600 text-slate-300 hover:bg-slate-700'}`}><Trash2 size={14}/> {sellMode?'SELLING':'SCRAP'}</button>
            </div>
            <div className="w-full max-w-3xl mb-2 flex gap-2 justify-center bg-slate-900/50 p-1.5 rounded-xl backdrop-blur-sm border border-white/5">
              <div className="flex flex-1 gap-1">
                {['ALL','R','P','S','N'].map(f => <button key={f} onClick={safeAction(()=>setDeckFilter(f))} className={`flex-1 py-2 sm:py-2.5 text-[10px] sm:text-xs font-black rounded-lg transition-all ${deckFilter===f?(f==='R'?'bg-stone-500 shadow-[0_0_10px_gray]':f==='P'?'bg-emerald-500 shadow-[0_0_10px_green]':f==='S'?'bg-orange-500 shadow-[0_0_10px_orange]':f==='N'?'bg-indigo-500 shadow-[0_0_10px_indigo]':'bg-cyan-500 shadow-[0_0_10px_cyan]')+' text-white scale-105 border border-white/40':'bg-slate-800/80 text-slate-400 hover:bg-slate-700 border border-transparent'}`}>{f==='ALL'?'ALL':MOVES[f]}</button>)}
              </div>
              <button onClick={safeAction(()=>setDeckSort(s=>s==='LVL'?'PWR':s==='PWR'?'ELM':'LVL'))} className="bg-slate-800 text-slate-300 text-[10px] px-3 rounded-lg font-black border border-slate-600 hover:bg-slate-700 flex items-center gap-1"><ListOrdered size={14}/> {deckSort}</button>
            </div>
            <div className="w-full max-w-3xl flex-1 flex flex-col min-h-0 bg-slate-900/40 rounded-3xl border border-white/5 shadow-inner pt-2">
              {renderDeck(handleDeckCardClick)}
            </div>
          </div>
        )}

        {view === 'LAB' && (
          <div className="h-full flex flex-col p-4 items-center relative z-10">
            <div className="w-full max-w-xl text-center mb-6 mt-2"><h2 className="text-4xl font-black text-pink-500 drop-shadow-[0_0_15px_rgba(236,72,153,0.8)] flex items-center justify-center gap-3 tracking-widest uppercase"><FlaskConical size={36}/> Mutant Lab</h2></div>
            <div className="flex justify-center gap-4 sm:gap-8 items-center mb-6 bg-slate-900/80 backdrop-blur-xl p-6 sm:p-8 rounded-[40px] border border-white/10 shadow-2xl w-full max-w-xl relative">
              <div className="w-28 sm:w-40"><CardView card={c1Preview} onClick={()=>togLab(fusionSlots[0])} /></div>
              <div className="flex flex-col items-center"><div className="text-4xl sm:text-5xl font-black text-cyan-400 drop-shadow-[0_0_15px_cyan] z-10 bg-slate-950 rounded-full w-14 h-14 sm:w-20 sm:h-20 flex items-center justify-center border-4 border-slate-800 shadow-inner">+</div></div>
              <div className="w-28 sm:w-40"><CardView card={c2Preview} onClick={()=>togLab(fusionSlots[1])} /></div>
            </div>
            {c1Preview && c2Preview && (
              <div className={`w-full max-w-xl bg-slate-900/90 backdrop-blur-md rounded-2xl p-4 mb-6 text-center shadow-[0_0_30px_rgba(6,182,212,0.2)] animate-slide-in relative overflow-hidden ${cityEvent.id==='BLOODMOON'?'border-rose-500/50':'border-cyan-500/50'}`}>
                <div className={`absolute top-0 left-0 w-full h-[2px] animate-[pulse_1s_infinite] ${cityEvent.id==='BLOODMOON'?'bg-rose-500':'bg-cyan-400'}`} />
                <p className={`text-[10px] font-bold mb-1 tracking-widest flex items-center justify-center gap-1 ${cityEvent.id==='BLOODMOON'?'text-rose-400':'text-cyan-400'}`}><Scan size={12}/> DNA PREDICTION</p>
                <p className={`text-xl font-black uppercase tracking-tight ${cityEvent.id==='BLOODMOON'?'text-rose-500':'text-pink-400'}`}>{predictedRecipe?.name} Class</p>
                <p className="text-sm text-slate-300 italic mb-2">{predictedRecipe?.desc}</p>
                <div className="bg-red-950 border border-red-700 text-red-400 text-[10px] font-black uppercase py-1 px-2 rounded flex items-center justify-center gap-1 w-max mx-auto"><AlertTriangle size={12}/> Parents Consumed!</div>
              </div>
            )}
            {blessedFusion && <div className="text-emerald-400 font-black text-xs mb-2 animate-pulse flex items-center gap-1"><Star size={14}/> Fortune Teller's Blessing (0% Curse Risk)</div>}
            <ActionBtn onClick={safeAction(handleFuse)} disabled={!fusionSlots[0]||!fusionSlots[1]||coins<100} className={`w-full max-w-md text-xl mb-4 ${(!fusionSlots[0]||!fusionSlots[1])?'bg-slate-800 text-slate-600 border-slate-700':'bg-pink-600 text-white shadow-[0_4px_0_#9d174d] active:shadow-none animate-pulse-fast ring-2 ring-pink-400'}`}>INITIATE FUSION (100 🪙)</ActionBtn>
            <div className="w-full max-w-md bg-slate-900 rounded-t-2xl p-2 border-t-2 border-x-2 border-slate-800 flex-1 overflow-hidden flex flex-col">
              <p className="text-center text-xs font-bold text-slate-500 mb-2 uppercase tracking-widest">Select Lvl 2+ Subjects</p>
              {renderDeck(togLab, true)}
            </div>
          </div>
        )}

        {view === 'BATTLE_PREP' && (
          <div className="flex flex-col h-full p-4 overflow-y-auto pb-24 items-center text-white relative">
            <div className="absolute top-0 left-0 w-full h-[600px] bg-[radial-gradient(ellipse_at_top,rgba(30,58,138,0.4)_0%,transparent_70%)] pointer-events-none" />
            <div className="w-full max-w-xl mb-8 flex justify-between items-center bg-blue-950/40 backdrop-blur-md p-3 rounded-2xl border border-blue-500/30 shadow-lg">
              {[1,2,3,4].map(r => {
                const cur = battleState?.streak || 1;
                return (
                  <div key={r} className={`flex-1 flex flex-col items-center relative transition-all duration-500 ${cur===r?'opacity-100 scale-125 z-10':cur>r?'opacity-60':'opacity-30'}`}>
                    {r < 4 && <div className={`absolute top-1/2 left-[50%] w-full h-1.5 rounded-full ${cur>r?'bg-blue-400 shadow-[0_0_10px_blue]':'bg-blue-900/50'}`} />}
                    <div className={`w-10 h-10 sm:w-12 sm:h-12 rounded-full flex items-center justify-center font-black z-10 border-2 transition-all ${cur===r?'bg-blue-500 text-white border-blue-200 shadow-[0_0_20px_blue]':cur>r?'bg-blue-900 text-blue-300 border-blue-700':'bg-slate-800 text-slate-500 border-slate-700'}`}>{r===4?<Skull size={18}/>:r}</div>
                  </div>
                );
              })}
            </div>
            <h2 className={`text-4xl sm:text-5xl font-black drop-shadow-[0_0_20px_rgba(59,130,246,0.8)] mb-8 uppercase italic tracking-tighter flex items-center gap-3 z-10 ${(battleState?.streak||1)>=4?'text-red-500 animate-pulse drop-shadow-[0_0_15px_red]':'text-blue-400'}`}><Trophy size={40}/> {battleState?.stage?.name || TOURNAMENT_STAGES[0].name}</h2>
            {(() => {
              const rival = battleState?.rival || ARENA_RIVALS[0][0];
              return (
                <div className="bg-slate-900/90 backdrop-blur-xl border border-white/10 rounded-3xl p-5 w-full max-w-xl z-10 mb-8 flex items-center gap-5 shadow-2xl">
                  <div className="text-6xl sm:text-7xl drop-shadow-[0_5px_10px_rgba(0,0,0,0.8)] bg-slate-950 p-4 rounded-2xl border border-white/5">{rival.avatar}</div>
                  <div className="flex-1">
                    <p className="text-[10px] text-slate-400 font-black tracking-widest uppercase bg-slate-950 inline-block px-3 py-1 rounded-full mb-1">UP NEXT</p>
                    <h3 className={`text-2xl sm:text-3xl font-black ${rival.color} drop-shadow-md tracking-tight leading-none mb-2`}>{rival.name}</h3>
                    <p className="text-sm text-slate-300 font-bold leading-snug bg-slate-800/50 p-2 rounded-lg border border-white/5">"{rival.desc}"</p>
                  </div>
                </div>
              );
            })()}
            {(battleState?.streak || 1) > 1 ? (
              <div className={`border-4 p-6 rounded-3xl mb-8 text-center w-full max-w-xl z-10 shadow-2xl relative overflow-hidden ${(battleState?.streak||1)>=4?'bg-red-950/90 border-red-500 shadow-[0_0_40px_rgba(220,38,38,0.5)] text-white':'bg-yellow-600/90 border-yellow-400 shadow-[0_0_40px_rgba(234,179,8,0.3)] text-black backdrop-blur-md'}`}>
                <div className="absolute inset-0 bg-[radial-gradient(circle_at_center,rgba(255,255,255,0.2)_0%,transparent_70%)] animate-pulse pointer-events-none" />
                <p className={`font-black text-sm tracking-widest relative z-10 ${(battleState?.streak||1)>=4?'text-red-300':'text-yellow-900'}`}>CURRENT POT</p>
                <p className="font-black text-7xl sm:text-8xl drop-shadow-lg relative z-10">{battleState?.pot} 🪙</p>
                {(battleState?.streak||1)>=4 && <p className="text-sm font-black text-red-200 mt-4 uppercase tracking-widest animate-pulse bg-red-900/80 inline-block px-6 py-2 rounded-full border border-red-500 relative z-10 shadow-lg">Winner Takes All. 1 Round Only.</p>}
              </div>
            ) : (
              <div className="bg-slate-900/80 backdrop-blur-md border border-white/10 p-5 rounded-3xl mb-8 w-full max-w-xl z-10 shadow-xl">
                <p className="text-center text-xs font-black text-cyan-400 mb-3 tracking-widest uppercase QUALIFIERS WAGER">QUALIFIERS WAGER</p>
                <div className="flex gap-3">{[10,50,100].map(w=><button key={w} onClick={safeAction(()=>setWager(w))} className={`flex-1 py-4 text-xl sm:text-2xl font-black rounded-2xl transition-all ${wager===w?'bg-blue-500 text-white scale-105 shadow-[0_0_20px_rgba(59,130,246,0.6)] ring-2 ring-white border-blue-300':'bg-slate-800 text-slate-400 hover:bg-slate-700 border border-slate-600'}`}>{w} 🪙</button>)}</div>
              </div>
            )}
            <div className="w-full max-w-xl bg-slate-900/90 backdrop-blur-xl border border-white/10 p-5 sm:p-6 rounded-[40px] mb-8 relative z-10 shadow-2xl">
              <div className="absolute -top-4 left-1/2 -translate-x-1/2 bg-blue-600 text-white text-xs font-black px-6 py-1.5 rounded-full border border-blue-300 tracking-widest shadow-[0_0_15px_rgba(37,99,235,0.5)]">YOUR LINEUP</div>
              <div className="flex justify-between items-center mt-4 gap-2 sm:gap-4">
                {teamIds.slice(0,3).map((id,i) => <div key={i} className={`flex-1 flex flex-col items-center transition-all ${(battleState?.streak||1)>=4&&i>0?'opacity-20 grayscale scale-90':''}`}><CardView card={getCard(id)} onClick={()=>id&&toggleTeam(id)}/><span className="text-[10px] sm:text-xs font-black text-slate-500 mt-2 bg-slate-950 px-3 py-1 rounded-full border border-slate-800">F{i+1}</span></div>)}
              </div>
              {activeSynergy && (
                <div className={`mt-4 text-center text-xs sm:text-sm px-4 py-2 rounded-xl border-2 font-black flex flex-col sm:flex-row items-center justify-center gap-2 shadow-lg ${SYNERGIES[activeSynergy].color}`}>
                  <div className="flex items-center gap-1"><SynergyIcon size={16}/> <span>{SYNERGIES[activeSynergy].name}:</span></div>
                  <span className="opacity-90">{SYNERGIES[activeSynergy].desc}</span>
                </div>
              )}
              <div className="bg-slate-950/50 border border-slate-800/50 rounded-2xl p-2 mt-4 flex items-center justify-between shadow-inner">
                <div className="flex flex-col px-2">
                  <span className="bg-amber-500 text-black text-[10px] font-black px-3 py-0.5 rounded-full border border-amber-200 shadow-md w-max mb-1">SUPPORT</span>
                  <span className="text-[10px] sm:text-xs text-slate-400 font-bold leading-tight">Passive ability shared with team.</span>
                </div>
                <div className="w-[60px] sm:w-[72px] shrink-0"><CardView card={getCard(teamIds[3])} small onClick={()=>toggleTeam(teamIds[3])}/></div>
              </div>
            </div>
            <div className="w-full max-w-xl mt-auto z-10 flex flex-col bg-slate-900/60 backdrop-blur-md pt-4 px-2 sm:px-4 rounded-t-[40px] border-t border-x border-white/10 shadow-[0_-10px_30px_rgba(0,0,0,0.5)] min-h-[350px] max-h-[55vh] overflow-hidden">
              <div className="mb-2 pb-4 border-b border-white/10 shrink-0">
                <ActionBtn onClick={safeAction(()=>{ if(battleState&&battleState.streak>1) startArena(true,battleState.pot,battleState.streak); else startArena(false,0,1); })} disabled={teamIds.slice(0,3).includes(null)} className={`w-full text-2xl py-5 shadow-[0_6px_0_#1e3a8a] border-blue-400 ${teamIds.slice(0,3).includes(null)?'bg-slate-800 text-slate-500 shadow-[0_4px_0_#0f172a] border-slate-700':'bg-blue-600 text-white active:shadow-none ring-4 ring-blue-500/50'}`}>
                  {teamIds.slice(0,3).includes(null)?'SELECT 3 FIGHTERS':(battleState&&battleState.streak>1?`START ROUND ${battleState.streak}`:'ENTER ARENA')}
                </ActionBtn>
              </div>
              <p className="text-center text-xs font-black text-cyan-400 mb-2 tracking-widest animate-pulse uppercase shrink-0">TAP CARDS BELOW TO EQUIP</p>
              {renderDeck(toggleTeam)}
            </div>
          </div>
        )}

        {view === 'BATTLE_PLAY' && battleState && (
          <div className={`flex flex-col h-full items-center text-white relative p-4 overflow-hidden ${battleState.phase==='BUSTED'?'scanlines animate-glitch':''} ${battleState.accused==='SUCCESS'?'bg-yellow-900/30':''}`}>
            <div className={`absolute top-0 left-0 w-full h-1.5 z-50 transition-colors duration-300 ${isRiot?'bg-red-500 shadow-[0_0_30px_red]':battleState.isSuddenDeath||battleState.isUltimateDouble?'bg-red-500 shadow-[0_0_20px_red]':'bg-gradient-to-r from-blue-400 via-purple-400 to-red-400 shadow-[0_0_15px_rgba(255,255,255,0.5)]'}`} />

            <div className={`w-full max-w-xl mb-2 mt-1 px-2 z-30 transition-all ${isRiot?'animate-pulse scale-105':''}`}>
              <div className="flex justify-between items-end mb-1">
                <span className={`text-[10px] font-black tracking-widest flex items-center gap-1 ${isRiot?'text-red-400':'text-amber-400'}`}><Flame size={12} className={isRiot?'animate-bounce':''}/> CROWD HYPE</span>
                <span className={`text-xs font-black ${isRiot?'text-red-500':'text-white'}`}>{isRiot?'RIOT!':`${crowdHype}%`}</span>
              </div>
              <div className="w-full h-2 bg-slate-900 rounded-full overflow-hidden border border-white/10 shadow-inner">
                <div className={`h-full transition-all duration-500 ${isRiot?'bg-red-500 shadow-[0_0_10px_red]':'bg-gradient-to-r from-amber-600 to-amber-400'}`} style={{width:`${isRiot?100:crowdHype}%`}} />
              </div>
            </div>

            <div className={`w-full max-w-xl flex flex-col gap-2 z-30 mb-8 transition-transform ${battleState.isRigged&&battleState.phase!=='PREDICT'?'rotate-1 opacity-95':''}`}>
              <div className={`flex justify-between items-center w-full bg-slate-900/80 backdrop-blur-md p-2 rounded-2xl border shadow-lg ${isRiot?'border-red-500 shadow-[0_0_20px_rgba(239,68,68,0.3)]':'border-white/10'}`}>
                <div className="bg-blue-900/80 border border-blue-400 px-4 py-2 rounded-xl font-black text-lg shadow-inner flex flex-col items-center leading-none"><span className="text-[10px] text-blue-300">YOU</span><span>{battleState.pScore}</span></div>
                <div className={`bg-yellow-500 text-black border-2 border-yellow-200 px-8 py-2 rounded-2xl font-black text-2xl shadow-[0_4px_0_#a16207] flex flex-col items-center leading-none ${battleState.isSuddenDeath||battleState.isUltimateDouble?'bg-red-600 text-white border-red-300 shadow-[0_4px_0_#7f1d1d] animate-pulse':''}`}>
                  <span className="text-[10px] tracking-widest opacity-80 mb-0.5">POT</span><span>{battleState.pot}</span>
                </div>
                <div className={`bg-red-900/80 border border-red-400 px-4 py-2 rounded-xl font-black text-lg shadow-inner flex items-center gap-3 ${battleState.isRigged?'animate-pulse text-red-300':''}`}>
                  <div className="flex flex-col items-center leading-none"><span className="text-[10px] text-red-300">{battleState.isRigged?'C?U':'CPU'}</span><span>{battleState.eScore}</span></div>
                  <span className="text-3xl drop-shadow-md bg-black/50 p-1 rounded-lg">{battleState.rival?.avatar}</span>
                </div>
              </div>
              <div className={`bg-slate-900/90 backdrop-blur-md rounded-2xl p-3 flex flex-col relative overflow-hidden shadow-xl border ${isRiot?'border-red-500/50 opacity-50 grayscale':'border-white/10'}`}>
                <div className="flex justify-between text-[10px] sm:text-xs font-black text-slate-400 mb-1.5 relative z-10 tracking-widest">
                  <span className="flex items-center gap-1.5"><Eye size={14}/> {isRiot?'JUDGES OVERRUN':'JUDGE SUSPICION'} {relics.includes('r2')&&!isRiot&&<span className="text-emerald-400 bg-emerald-950/50 px-2 rounded">(BRIBED)</span>} {relics.includes('r4')&&!isRiot&&<span className="text-amber-400 bg-amber-950/50 px-2 rounded">(CLEAN)</span>}</span>
                  <span className="text-white bg-slate-950 px-2 py-0.5 rounded">{isRiot?'OFF':`${suspicion}%`}</span>
                </div>
                <div className="w-full h-3 bg-black rounded-full overflow-hidden relative z-10 border border-slate-700 shadow-inner">
                  <div className={`h-full transition-all duration-500 rounded-full ${suspicion>=maxSuspicion*0.7?'bg-red-500 shadow-[0_0_10px_red] animate-pulse':'bg-gradient-to-r from-orange-500 to-red-500'}`} style={{width:`${isRiot?0:Math.min(100,(suspicion/maxSuspicion)*100)}%`}} />
                </div>
                {suspicion>=maxSuspicion*0.7&&!isRiot&&<div className="absolute inset-0 bg-red-600/10 animate-ping z-0" />}
              </div>
              <div className="flex flex-wrap gap-2 w-full justify-center">
                {battleState.modifier?.id !== 'NORMAL' && <div className={`text-[10px] sm:text-xs font-black px-3 py-1.5 rounded-lg border shadow-lg flex items-center gap-1.5 ${battleState.modifier.color}`}>{battleState.isSuddenDeath||battleState.isUltimateDouble?<Skull size={14}/>:<Hexagon size={14}/>} {battleState.modifier.name}</div>}
                {battleState.synergy && <div className={`text-[10px] sm:text-xs font-black px-3 py-1.5 rounded-lg border shadow-lg flex items-center gap-1.5 ${SYNERGIES[battleState.synergy].color}`}><SynergyIcon size={14}/> {SYNERGIES[battleState.synergy].name}</div>}
              </div>
            </div>

            {battleState.phase === 'ULTIMATE_OFFER' && (
              <div className="w-full flex-1 flex flex-col items-center justify-center animate-slide-in space-y-6 relative z-20 px-4">
                <div className="absolute inset-0 bg-[radial-gradient(circle_at_center,rgba(220,38,38,0.2)_0%,transparent_70%)] animate-pulse z-0 pointer-events-none" />
                <h2 className="text-6xl sm:text-7xl font-black text-red-500 drop-shadow-[0_0_20px_red] text-center z-10 leading-none tracking-tighter italic">THE ULTIMATE<br/>DOUBLE</h2>
                <p className="text-center font-bold text-slate-300 text-sm sm:text-base z-10 max-w-sm bg-black/50 p-4 rounded-xl border border-red-900/50 backdrop-blur-sm">Beat the Boss. Walk away... or risk it all for <span className="text-yellow-400 font-black">x5 the Pot</span>.</p>
                <div className="w-full max-w-sm space-y-4 z-10 mt-8">
                  <ActionBtn onClick={safeAction(handleCashOut)} disabled={inputLocked} className="w-full bg-emerald-600 text-white text-2xl py-5 shadow-[0_6px_0_#047857] active:shadow-none flex flex-col justify-center items-center border border-emerald-400">
                    <span className="font-black text-2xl sm:text-3xl">CASH OUT & END</span>
                    <span className="text-emerald-200 text-xs sm:text-sm mt-1 font-bold tracking-widest bg-emerald-950/60 px-4 py-1 rounded-full shadow-inner">Claim +{battleState.pot} 🪙</span>
                  </ActionBtn>
                  <div className="text-center font-black text-slate-500 text-xs tracking-widest my-2">- OR -</div>
                  <ActionBtn onClick={safeAction(startUltimateDouble)} disabled={inputLocked} className="w-full bg-red-800 text-yellow-400 text-3xl py-6 shadow-[0_6px_0_#450a0a] active:shadow-none flex flex-col items-center animate-pulse-fast border border-red-500 ring-4 ring-red-900/50">
                    <div className="flex items-center gap-2 font-black tracking-tighter"><Skull size={36}/> RISK IT ALL</div>
                    <span className="text-sm font-bold text-white mt-2 bg-red-950 px-4 py-1 rounded-lg border border-red-700 shadow-inner">For {battleState.pot*5} 🪙</span>
                  </ActionBtn>
                </div>
              </div>
            )}

            {battleState.phase === 'BUSTED' && (
              <div className="w-full flex-1 flex flex-col items-center justify-center animate-slide-in space-y-6 relative z-20">
                <div className="absolute inset-0 bg-red-900/20 backdrop-blur-sm z-0" />
                <Gavel size={120} className="text-red-500 animate-[bounce_0.5s_infinite] drop-shadow-[0_0_40px_rgba(239,68,68,1)] z-10"/>
                <h2 className="text-[100px] leading-none font-black text-red-600 drop-shadow-[0_10px_0_#450a0a] tracking-tighter z-10 text-center italic">BUSTED!</h2>
                <p className="text-center font-bold text-red-200 text-xl z-10 bg-black/80 p-6 rounded-2xl border-2 border-red-900 shadow-2xl">The Judges caught you cheating.<br/><span className="text-white">All coins seized.</span></p>
                <ActionBtn onClick={safeAction(()=>endRun(true))} disabled={inputLocked} className="w-full max-w-xs bg-red-700 text-white text-3xl px-12 py-6 mt-8 shadow-[0_8px_0_#7f1d1d] border-red-400 active:shadow-none z-10">Accept Fate</ActionBtn>
              </div>
            )}

            {battleState.phase === 'SELECT_FIGHTER' && (
              <div className="w-full max-w-xl flex-1 flex flex-col items-center justify-center relative animate-slide-in">
                <div className="flex flex-col items-center mb-8 relative">
                  <div className="absolute -inset-10 bg-[radial-gradient(circle_at_center,rgba(234,179,8,0.15)_0%,transparent_70%)] animate-pulse pointer-events-none" />
                  <h3 className="text-sm font-black text-amber-400 mb-4 tracking-widest bg-amber-950/50 px-4 py-1 rounded-full border border-amber-500/30 z-10">ENEMY DEPLOYED</h3>
                  <div className="w-32 sm:w-40 z-10"><CardView card={enemyTeam[battleState.roundNum]} hideStats={(!teamScout&&battleState.stage?.hideStats)||battleState.isUltimateDouble} /></div>
                </div>
                <div className="mt-4 mb-8 bg-slate-900/80 backdrop-blur-md px-6 py-3 rounded-2xl border border-white/10 flex items-center justify-center gap-3 shadow-xl w-full max-w-md">
                  <span className="text-stone-400 text-xl">✊</span><span className="text-slate-500 text-xs font-black tracking-widest">BEATS</span><span className="text-orange-400 text-xl">✌️</span><span className="text-slate-500 text-xs font-black tracking-widest">BEATS</span><span className="text-emerald-400 text-xl">✋</span><span className="text-slate-500 text-xs font-black tracking-widest">BEATS</span><span className="text-stone-400 text-xl">✊</span>
                </div>
                <h3 className="text-sm font-black text-cyan-400 mb-4 tracking-widest uppercase">Select Your Counter</h3>
                <div className="flex gap-2 sm:gap-4 justify-center w-full px-2">
                  {teamIds.slice(0,3).map((id,i) => (
                    <div key={i} className="flex-1 flex flex-col items-center relative transition-transform hover:-translate-y-2">
                      <CardView card={getCard(id)} onClick={safeAction(()=>selectFighter(i))} isLabDisabled={battleState.usedFighters.includes(i)||inputLocked} />
                      {battleState.usedFighters.includes(i) && <div className="absolute inset-0 bg-black/70 backdrop-blur-[2px] flex items-center justify-center z-30 rounded-2xl"><span className="text-red-500 font-black text-sm sm:text-base rotate-[-15deg] border-2 border-red-500 bg-red-950 px-3 py-1 rounded-lg shadow-[0_0_15px_red]">USED</span></div>}
                    </div>
                  ))}
                </div>
              </div>
            )}

            {battleState.phase === 'PREDICT' && (
              <div className="w-full max-w-xl flex-1 flex flex-col items-center justify-center relative animate-slide-in" key={battleState.animKey}>
                <h3 className="text-3xl font-black text-amber-400 drop-shadow-[0_0_15px_rgba(245,158,11,0.8)] mb-2 animate-pulse tracking-widest uppercase italic">Call It!</h3>
                <p className="text-amber-200/80 text-sm mb-10 font-bold bg-amber-950/50 px-4 py-1.5 rounded-full border border-amber-500/30 shadow-inner">Predict their move: +20 PWR / +10 🪙</p>
                <div className="flex w-full justify-center gap-6 sm:gap-12 items-center px-4 mb-16 relative">
                  <div className="w-[110px] sm:w-[140px]"><CardView card={battleState.activePlayerCard} /></div>
                  <div className="text-5xl sm:text-6xl font-black italic text-slate-700 drop-shadow-md">VS</div>
                  <div className="w-[110px] sm:w-[140px]"><CardView card={battleState.activeEnemyCard} hideStats={(!teamScout&&battleState.stage?.hideStats)||battleState.isUltimateDouble} /></div>
                </div>
                <div className="w-full max-w-md mt-auto mb-6 px-4">
                  <div className="flex gap-3 w-full">
                    {['R','P','S'].map(m => (
                      <ActionBtn key={m} disabled={inputLocked} onClick={safeAction(()=>handlePrediction(m))} className="flex-1 bg-slate-800 text-6xl py-6 border-slate-600 shadow-[0_6px_0_#334155] active:shadow-none hover:bg-amber-700 hover:border-amber-400 hover:shadow-[0_6px_0_#92400e] transition-colors group">
                        <span className="drop-shadow-lg group-hover:scale-110 transition-transform block">{MOVES[m]}</span>
                      </ActionBtn>
                    ))}
                  </div>
                </div>
              </div>
            )}

            {['FIGHTING','REACTION','CLASH','REVEAL'].includes(battleState.phase) && (
              <div className="w-full max-w-xl flex-1 flex flex-col items-center justify-center relative" key={battleState.animKey}>
                <h3 className={`text-xl sm:text-2xl font-black mb-10 tracking-[0.3em] uppercase italic drop-shadow-lg ${battleState.phase==='CLASH'?'text-amber-500 animate-pulse drop-shadow-[0_0_15px_orange]':'text-slate-500'}`}>
                  {battleState.phase==='CLASH'?'⚡ RHYTHM CLASH ⚡':`MATCH ${battleState.roundNum+1} OF ${battleState.isSuddenDeath||battleState.isUltimateDouble?'1':'3'}`}
                </h3>
                <div className={`flex w-full justify-center gap-6 sm:gap-12 items-center px-4 ${battleState.phase==='CLASH'?'animate-clash mt-4':'mb-16'} relative`}>
                  {battleState.slashAnim && <div className={`absolute inset-0 z-[100] ${battleState.slashAnim}`} />}
                  {battleState.showText && <div className={`absolute top-[10%] left-1/2 -translate-x-1/2 z-[100] text-5xl sm:text-6xl font-black italic tracking-tighter ${battleState.showText.color} drop-shadow-[0_10px_20px_black] animate-floatUp pointer-events-none whitespace-nowrap`}>{battleState.showText.txt}</div>}

                  <div className="w-[110px] sm:w-[140px] relative">
                    <CardView card={battleState.activePlayerCard} isFighting={battleState.phase==='FIGHTING'} />
                    {battleState.phase==='FIGHTING'&&battleState.activeEnemyCard?.id==='b1' && <div className="absolute -inset-6 bg-red-600/30 backdrop-blur-md z-40 flex items-center justify-center border-4 border-red-500 rounded-3xl animate-glitch shadow-[0_0_30px_red]"><span className="text-red-100 text-xl sm:text-2xl font-black rotate-[-15deg] bg-red-950/90 px-4 py-2 border-2 border-red-400 rounded-xl shadow-2xl tracking-widest">JAMMED</span></div>}
                    {battleState.phase==='FIGHTING' && !battleState.bluffActive && <div className="absolute inset-0 flex items-center justify-center text-7xl sm:text-8xl animate-fight z-30 drop-shadow-[0_10px_20px_rgba(0,0,0,0.8)]">🤜</div>}
                    {battleState.phase==='FIGHTING' && battleState.bluffActive && <div className="absolute inset-0 flex items-center justify-center text-7xl sm:text-8xl animate-spin z-30 drop-shadow-[0_10px_20px_rgba(0,0,0,0.8)] opacity-60">🃏</div>}
                    {battleState.phase==='REVEAL' && <div className="absolute inset-0 flex items-center justify-center text-[100px] sm:text-[120px] bg-slate-100 rounded-2xl z-40 border-8 border-slate-900 animate-flip shadow-[0_0_40px_rgba(255,255,255,0.8)] leading-none pt-4 pb-2">{MOVES[battleState.matches[battleState.roundNum]?.pM]}</div>}
                    {battleState.phase==='CLASH' && <div className="absolute -top-6 -right-6 sm:-right-8 bg-amber-500 text-black font-black text-4xl sm:text-5xl w-20 h-20 sm:w-24 sm:h-24 rounded-full flex items-center justify-center border-4 border-amber-100 z-40 shadow-[0_0_30px_orange]">{(battleState.activePlayerCard?.pwr||0)+(battleState.clashData?.pPwrBonus||0)}</div>}
                    {battleState.prediction && battleState.phase!=='REVEAL' && <div className="absolute -bottom-5 left-1/2 -translate-x-1/2 bg-amber-500 text-black text-[10px] sm:text-xs font-black px-3 py-1 rounded-full border-2 border-amber-200 z-40 shadow-lg whitespace-nowrap">CALLED: {MOVES[battleState.prediction]}</div>}
                  </div>

                  {battleState.phase==='CLASH' ? <div className="text-7xl sm:text-8xl z-30 animate-pulse text-amber-400 drop-shadow-[0_0_40px_#b45309]">⚡</div> : <div className="text-5xl sm:text-6xl font-black italic text-slate-700 drop-shadow-md">VS</div>}

                  <div className="w-[110px] sm:w-[140px] relative">
                    <CardView card={battleState.activeEnemyCard} hideStats={(!teamScout&&battleState.stage?.hideStats)||battleState.isUltimateDouble} isFighting={battleState.phase==='FIGHTING'} />
                    {battleState.phase==='FIGHTING'&&battleState.activeEnemyCard?.id==='b2' && <div className="absolute -top-14 left-1/2 -translate-x-1/2 bg-purple-900 text-purple-200 text-xs sm:text-sm font-black px-3 py-1 rounded-full border-2 border-purple-400 animate-pulse whitespace-nowrap z-50 shadow-[0_0_20px_purple]">👁️ SEES {getDominantStat(battleState.activeEnemyCard)}</div>}
                    {battleState.isRigged&&battleState.phase==='FIGHTING' && <div className="absolute inset-0 bg-[repeating-linear-gradient(0deg,transparent,transparent_2px,rgba(255,0,0,0.15)_2px,rgba(255,0,0,0.15)_4px)] z-40 pointer-events-none opacity-80 mix-blend-screen rounded-2xl shadow-[inset_0_0_30px_rgba(255,0,0,0.5)]" />}
                    {battleState.phase==='FIGHTING' && <div className="absolute inset-0 flex items-center justify-center text-7xl sm:text-8xl animate-fight z-30 drop-shadow-[0_10px_20px_rgba(0,0,0,0.8)]" style={{transform:'scaleX(-1)'}}>🤜</div>}
                    {battleState.phase==='REVEAL' && <div className="absolute inset-0 flex items-center justify-center text-[100px] sm:text-[120px] bg-slate-100 rounded-2xl z-40 border-8 border-slate-900 animate-flip shadow-[0_0_40px_rgba(255,255,255,0.8)] leading-none pt-4 pb-2">{MOVES[battleState.matches[battleState.roundNum]?.eM]}</div>}
                    {battleState.phase==='CLASH' && <div className="absolute -top-6 -left-6 sm:-left-8 bg-red-600 text-white font-black text-4xl sm:text-5xl w-20 h-20 sm:w-24 sm:h-24 rounded-full flex items-center justify-center border-4 border-red-200 z-40 shadow-[0_0_30px_red]">{(battleState.activeEnemyCard?.pwr||0)+(battleState.clashData?.ePwrBonus||0)}</div>}
                    {battleState.phase==='REACTION' && (battleState.readSuccess||battleState.accused==='SUCCESS') && (
                      <>
                        <div className="absolute inset-0 flex items-center justify-center text-8xl sm:text-9xl z-30 drop-shadow-[0_0_30px_cyan] opacity-95 animate-pulse bg-black/60 rounded-2xl backdrop-blur-md">{MOVES[battleState.pendingMoves?.eM]}</div>
                        <div className="absolute -top-6 left-1/2 -translate-x-1/2 text-cyan-300 font-black text-xs sm:text-sm animate-bounce drop-shadow-[0_0_10px_cyan] whitespace-nowrap bg-cyan-950/80 border border-cyan-400 px-3 py-1 rounded-full shadow-lg z-40">CRITICAL READ!</div>
                      </>
                    )}
                  </div>
                </div>

                {battleState.phase==='CLASH' && (() => {
                  const pTotal = (battleState.activePlayerCard?.pwr||0)+(battleState.clashData?.pPwrBonus||0);
                  const eTotal = (battleState.activeEnemyCard?.pwr||0)+(battleState.clashData?.ePwrBonus||0);
                  const ratio = Math.max(5, Math.min(95, (pTotal/Math.max(1,pTotal+eTotal))*100));
                  return (
                    <div className="w-full mt-6 flex flex-col items-center">
                      <div className="w-full max-w-sm h-12 bg-slate-900 rounded-full flex overflow-hidden border-4 border-slate-700 relative shadow-[0_10px_30px_rgba(0,0,0,0.8)]">
                        <div className="h-full bg-blue-600 transition-all duration-100 flex items-center px-6 font-black text-white text-2xl sm:text-3xl shadow-[inset_-10px_0_20px_rgba(0,0,0,0.3)]" style={{width:`${ratio}%`}}>YOU</div>
                        <div className="h-full bg-red-600 transition-all duration-100 flex items-center justify-end px-6 font-black text-white text-2xl sm:text-3xl shadow-[inset_10px_0_20px_rgba(0,0,0,0.3)]" style={{width:`${100-ratio}%`}}>FOE</div>
                        <div className="absolute left-1/2 top-0 bottom-0 w-3 bg-yellow-400 -translate-x-1/2 z-10 shadow-[0_0_15px_yellow]" />
                      </div>
                    </div>
                  );
                })()}

                {battleState.phase==='REVEAL' && (() => {
                  const m = battleState.matches[battleState.roundNum];
                  return <div className={`text-[80px] text-center leading-none tracking-tighter font-black absolute top-[40%] left-1/2 -translate-x-1/2 -translate-y-1/2 z-40 drop-shadow-[0_10px_10px_rgba(0,0,0,0.8)] ${m?.res===1?'text-emerald-400':m?.res===-1?'text-red-500':'text-slate-400'}`}>{m?.res===1?(m.wasClash?'OVERPOWER!':'WIN!'):m?.res===-1?(m.wasClash?'SHATTERED!':'LOSE!'):'DRAW'}</div>;
                })()}

                <div className="w-full mt-auto mb-2 relative flex flex-col items-center justify-end z-20 pb-safe max-w-md">
                  {battleState.phase==='FIGHTING' && (
                    <>
                      {battleState.shuffledMoves?.join('')!=='RPS' && <div className="text-purple-300 font-black text-xs sm:text-sm mb-2 animate-pulse tracking-widest bg-purple-950/80 px-4 py-1.5 rounded-full border border-purple-400 shadow-[0_0_15px_purple]">👁️ ILLUSION: BUTTONS SCRAMBLED</div>}
                      <div className="flex gap-2 w-full px-2 mb-2">
                        {!relics.includes('r4') && battleState.activeEnemyCard?.id!=='b1' && battleState.activeEnemyCard?.id!=='warden'
                          ? (battleState.shuffledMoves||['R','P','S']).map(m => (
                              <ActionBtn key={m} disabled={inputLocked || battleState.cheatedThisMatch} onClick={safeAction(()=>handleCheat(m))} className="flex-1 bg-slate-800 text-4xl sm:text-5xl py-4 sm:py-5 border-slate-600 shadow-[0_6px_0_#334155] active:shadow-none hover:bg-slate-700 overflow-hidden group">
                                <span className="relative z-10 group-hover:scale-110 transition-transform block drop-shadow-md">{MOVES[m]}</span>
                                <div className="absolute inset-0 bg-red-600/40 translate-y-full group-hover:translate-y-0 transition-transform" />
                              </ActionBtn>
                            ))
                          : relics.includes('r4')
                            ? <div className="text-emerald-500 font-black tracking-widest bg-emerald-900/40 px-6 py-6 rounded-2xl border-2 border-emerald-500 w-full text-center shadow-[0_0_20px_rgba(16,185,129,0.3)] text-xl flex items-center justify-center">📜 CLEAN RECORD ACTIVE</div>
                            : <div className="text-red-400 font-black tracking-widest bg-red-900/40 px-6 py-6 rounded-2xl border-2 border-red-500 animate-pulse w-full text-center shadow-[0_0_20px_rgba(239,68,68,0.5)] text-xl flex items-center justify-center gap-3"><ShieldAlert size={28}/> SIGNAL JAMMED</div>
                        }
                      </div>
                      
                      {battleState.activePlayerCard?.heat >= 5 && !battleState.bluffActive && !relics.includes('r4') && battleState.activeEnemyCard?.id!=='warden' && (
                         <div className="w-full px-2 mb-2">
                           <ActionBtn onClick={safeAction(()=>handleCheat('N'))} disabled={inputLocked || battleState.cheatedThisMatch} className="w-full bg-indigo-600 text-white shadow-[0_4px_0_#3730a3] active:shadow-none border-indigo-400 py-3 animate-pulse ring-2 ring-indigo-300">
                             <span className="font-black tracking-widest flex items-center gap-2 text-xl"><Star size={20}/> NOVA STRIKE <Star size={20}/></span>
                           </ActionBtn>
                         </div>
                      )}

                      <div className="flex gap-2 w-full px-2 mb-3">
                        {!battleState.bluffActive && !relics.includes('r4') && battleState.activeEnemyCard?.id!=='b1' && battleState.activeEnemyCard?.id!=='warden' && (
                          <ActionBtn onClick={safeAction(handleBluff)} disabled={coins<15 || inputLocked || battleState.bluffActive} className="flex-1 bg-fuchsia-800 text-white text-sm py-3 border-fuchsia-600 shadow-[0_4px_0_#4a044e] active:shadow-none flex flex-col items-center justify-center hover:bg-fuchsia-700">
                            <span className="font-black tracking-widest">🃏 BLUFF</span><span className="text-[10px] text-fuchsia-200 mt-0.5 bg-fuchsia-950 px-2 py-0.5 rounded">-15 🪙</span>
                          </ActionBtn>
                        )}
                        {!battleState.accused && !relics.includes('r4') && (
                          <ActionBtn onClick={safeAction(handleAccuse)} disabled={inputLocked || battleState.accused} className="flex-1 bg-yellow-600 border-yellow-300 text-white shadow-[0_4px_0_#ca8a04] active:shadow-none flex flex-col items-center justify-center hover:bg-yellow-500 z-10">
                            <span className="font-black tracking-widest text-sm">⚖️ ACCUSE</span><span className="text-[10px] text-yellow-100 mt-0.5 bg-yellow-900 px-2 py-0.5 rounded">CALL JUDGE</span>
                          </ActionBtn>
                        )}
                      </div>
                      {tricks.length > 0 && !battleState.activeTrick && (
                        <div className="flex gap-2 w-full px-2 mb-3">
                          {tricks.map((tid,i) => { const t=TRICKS_DB.find(tr=>tr.id===tid); if(!t) return null; return <ActionBtn key={i} disabled={inputLocked} onClick={safeAction(()=>handleUseTrick(t))} className="flex-1 bg-cyan-900 text-cyan-200 border-cyan-500 py-2 flex items-center justify-center shadow-md hover:bg-cyan-800 hover:text-white text-2xl">{t.icon}</ActionBtn>; })}
                        </div>
                      )}
                    </>
                  )}

                  {battleState.phase==='REACTION' && (
                    <>
                      <div className="flex gap-2 w-full px-2 mb-2">
                        {(battleState.shuffledMoves||['R','P','S']).map(m => (
                          <ActionBtn key={m} disabled={inputLocked} onPointerDown={safeAction((e)=>{e.preventDefault();handleQuickSwap(m);})} className="flex-1 bg-cyan-900 text-5xl sm:text-6xl py-4 sm:py-5 border-cyan-500 shadow-[0_6px_0_#083344] active:shadow-none hover:bg-cyan-800 transition-transform z-10 group">
                            <span className="drop-shadow-lg block group-hover:-translate-y-1 transition-transform">{MOVES[m]}</span>
                          </ActionBtn>
                        ))}
                      </div>
                      {!battleState.accused && (
                        <div className="flex gap-2 w-full px-2 mb-3">
                          <ActionBtn disabled={inputLocked || battleState.accused} onClick={safeAction(handleAccuse)} className="flex-1 bg-yellow-600 border-yellow-300 text-white shadow-[0_4px_0_#ca8a04] active:shadow-none flex flex-col items-center justify-center hover:bg-yellow-500 z-10">
                            <span className="font-black tracking-widest text-sm">⚖️ ACCUSE</span><span className="text-[10px] text-yellow-100 mt-0.5 bg-yellow-900 px-2 py-0.5 rounded">CALL JUDGES</span>
                          </ActionBtn>
                        </div>
                      )}
                    </>
                  )}

                  {battleState.phase==='CLASH' && (
                    <div className="w-full px-4 mb-4 flex flex-col items-center">
                      <div className="text-xl font-black text-amber-400 drop-shadow-md mb-2">{(clashTime/10).toFixed(1)}s</div>
                      <button onPointerDown={handleMash} disabled={inputLocked} className={`w-full hover:bg-amber-400 text-black font-black text-4xl sm:text-5xl py-6 rounded-2xl border-4 active:shadow-none active:translate-y-2 transition-all flex justify-center items-center gap-4 select-none touch-manipulation relative overflow-hidden ${pulseSync?'bg-white border-yellow-300 shadow-[0_0_30px_white]':'bg-amber-500 border-amber-200 shadow-[0_10px_0_#92400e]'}`}>
                        <Crosshair size={32} className={pulseSync?'animate-spin':''}/> {pulseSync?'PERFECT!':'MASH!'}
                        {clashHitMarker && <span key={clashHitMarker.id} className="absolute left-[80%] top-1/2 -translate-y-1/2 text-3xl sm:text-4xl text-white font-black animate-floatUp pointer-events-none drop-shadow-[0_5px_5px_rgba(0,0,0,0.8)]">+{clashHitMarker.val}</span>}
                      </button>
                    </div>
                  )}

                  {(battleState.phase==='FIGHTING'||battleState.phase==='REACTION') && (
                    <div className="w-full px-2">
                      <div className="w-full h-2 bg-neutral-900 rounded-full overflow-hidden border border-neutral-800 shadow-inner">
                        <div className="h-full bg-cyan-400 w-full animate-[shrinkX_linear_forwards] shadow-[0_0_10px_cyan]" style={{animationDuration:`${battleState.reactionMs}ms`,transformOrigin:'left'}} />
                      </div>
                    </div>
                  )}
                </div>
              </div>
            )}

            {battleState.phase==='DONE' && (
              <div className="w-full max-w-md flex flex-col h-full pb-8 pt-16 items-center z-30 animate-slide-in">
                <h2 className={`text-[70px] sm:text-[80px] leading-none tracking-tighter font-black text-center drop-shadow-[0_10px_10px_rgba(0,0,0,1)] mb-8 italic ${battleState.pScore>battleState.eScore?'text-emerald-400':battleState.pScore===battleState.eScore?'text-slate-400':'text-red-500'}`}>{battleState.pScore>battleState.eScore?'VICTORY!':battleState.pScore===battleState.eScore?'DRAW!':'WIPEOUT!'}</h2>
                <div className="flex justify-center gap-4 sm:gap-6 mb-12 w-full px-4">
                  {battleState.matches.map((m,i) => (
                    <div key={i} className={`flex-1 flex flex-col items-center bg-slate-900/80 backdrop-blur-md border-4 rounded-3xl p-5 shadow-2xl ${m.res===1?'border-emerald-500 shadow-[0_0_20px_rgba(16,185,129,0.3)]':m.res===-1?'border-red-600 shadow-[0_0_20px_rgba(220,38,38,0.3)]':'border-slate-700'}`}>
                      <span className="text-5xl sm:text-6xl drop-shadow-md">{MOVES[m.pM]}</span>
                      <div className="w-full h-1 bg-slate-800 my-5 relative"><span className="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 text-[10px] font-black text-slate-400 bg-slate-950 px-2 py-0.5 rounded-full border border-slate-700">{m.wasClash?'⚡':'VS'}</span></div>
                      <span className="text-5xl sm:text-6xl drop-shadow-md">{MOVES[m.eM]}</span>
                    </div>
                  ))}
                </div>
                {battleState.pScore > battleState.eScore ? (
                  <div className="w-full space-y-4 px-4 max-w-sm">
                    <ActionBtn onClick={safeAction(handleCashOut)} disabled={inputLocked} className="w-full bg-emerald-600 text-white py-5 shadow-[0_6px_0_#047857] active:shadow-none flex flex-col justify-center items-center px-6 border border-emerald-400">
                      <span className="font-black text-2xl sm:text-3xl tracking-tight">CASH OUT & END</span>
                      <span className="text-emerald-200 text-xs sm:text-sm mt-1 font-bold tracking-widest bg-emerald-950/60 px-4 py-1 rounded-full shadow-inner">Claim +{battleState.pot} 🪙</span>
                    </ActionBtn>
                    {!battleState.isSuddenDeath && !battleState.isUltimateDouble && (
                      <>
                        <div className="text-center font-black text-slate-500 text-xs tracking-widest my-3">- OR -</div>
                        <ActionBtn onClick={safeAction(handleLetItRide)} disabled={inputLocked} className="w-full bg-yellow-500 text-black py-5 shadow-[0_6px_0_#a16207] active:shadow-none flex flex-col justify-center items-center px-6 border border-yellow-200 animate-pulse-fast ring-2 ring-yellow-400/50">
                          <span className="font-black text-2xl sm:text-3xl tracking-tight flex items-center gap-2"><FastForward size={28}/> LET IT RIDE</span>
                          <span className="text-yellow-900 text-xs sm:text-sm mt-1 font-bold tracking-widest bg-yellow-300/60 px-4 py-1 rounded-full shadow-inner">
                            Next Round → {Math.floor((battleState?.pot||0) * (teamGreed?1.5:1) * ((TOURNAMENT_STAGES[Math.min((battleState?.streak||1), TOURNAMENT_STAGES.length-1)]?.mult||2) / (battleState?.stage?.mult||1)))} 🪙
                          </span>
                        </ActionBtn>
                      </>
                    )}
                  </div>
                ) : (
                  <div className="w-full flex flex-col items-center gap-4 px-4 max-w-sm">
                    <ActionBtn onClick={safeAction(handleResetToArena)} disabled={inputLocked} className="w-full bg-slate-700 text-white py-4 shadow-[0_4px_0_#0f172a] active:shadow-none border-slate-500">
                      <span className="font-black text-xl">TRY AGAIN</span>
                    </ActionBtn>
                  </div>
                )}
              </div>
            )}
          </div>
        )}

      </main>

      {needsBottomNav && appState === 'PLAYING' && (
        <nav className="fixed bottom-0 left-0 right-0 z-40 bg-slate-900/95 backdrop-blur-xl border-t-2 border-white/10 flex items-stretch shadow-[0_-5px_30px_rgba(0,0,0,0.5)]">
          {[
            { id:'EXPLORE',     label:'CITY',   Icon: MapIcon,      color:'text-emerald-400' },
            { id:'DECK',        label:'ROSTER', Icon: Library,      color:'text-cyan-400'    },
            { id:'LAB',         label:'LAB',    Icon: FlaskConical, color:'text-pink-400'    },
            { id:'BATTLE_PREP', label:'ARENA',  Icon: Swords,       color:'text-red-400'     },
          ].map(({ id, label, Icon, color }) => (
            <button
              key={id}
              onClick={safeAction(() => handleNav(id))}
              className={`flex-1 flex flex-col items-center justify-center gap-1 py-3 transition-all relative active:scale-95 ${view === id ? color + ' bg-white/5' : 'text-slate-500 hover:text-slate-300'}`}
            >
              {view === id && (
                <div className="absolute top-0 left-1/2 -translate-x-1/2 w-10 h-1 rounded-b-full bg-current shadow-[0_0_8px_currentColor]" />
              )}
              <Icon size={22} strokeWidth={view === id ? 2.5 : 1.5} />
              <span className="text-[10px] font-black tracking-widest uppercase leading-none">{label}</span>
              {id === 'BATTLE_PREP' && (() => {
                const streak = battleState?.streak || 0;
                if (streak > 1) return (
                  <div className="absolute -top-2 right-3 bg-yellow-400 text-black text-[8px] font-black px-1.5 py-0.5 rounded-full border border-yellow-200 shadow-md animate-pulse leading-none">
                    x{streak}
                  </div>
                );
                return null;
              })()}
              {id === 'EXPLORE' && cityHeat >= 80 && (
                <div className="absolute -top-2 right-3 bg-red-500 text-white text-[8px] font-black px-1.5 py-0.5 rounded-full border border-red-200 shadow-md animate-bounce leading-none">
                  ⚠️
                </div>
              )}
            </button>
          ))}

          <div className="absolute -top-9 left-0 right-0 flex justify-between items-center px-4 pointer-events-none">
            <div className="bg-slate-900/90 backdrop-blur-md border border-white/10 text-yellow-400 font-black text-sm px-4 py-1.5 rounded-full shadow-lg flex items-center gap-2">
              <span className="text-yellow-300">🪙</span>
              <span>{coins.toLocaleString()}</span>
            </div>
            <button
              onClick={() => setIsMuted(!isMuted)}
              className="pointer-events-auto bg-slate-900/90 backdrop-blur-md border border-white/10 text-slate-400 hover:text-white p-2 rounded-full shadow-lg transition-colors active:scale-95"
            >
              {isMuted ? <VolumeX size={16} /> : <Volume2 size={16} />}
            </button>
          </div>
        </nav>
      )}

      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
        .bg-grid-pattern {
          background-image: linear-gradient(rgba(255,255,255,0.03) 1px, transparent 1px),
                            linear-gradient(90deg, rgba(255,255,255,0.03) 1px, transparent 1px);
          background-size: 40px 40px;
        }
        .scanlines::after {
          content: ''; position: fixed; top: 0; left: 0; right: 0; bottom: 0;
          background: repeating-linear-gradient(0deg, transparent, transparent 2px, rgba(0,0,0,0.15) 2px, rgba(0,0,0,0.15) 4px);
          pointer-events: none; z-index: 9999;
        }
        .perspective-1000 { perspective: 1000px; }
        .custom-scrollbar::-webkit-scrollbar { width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.1); border-radius: 2px; }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover { background: rgba(255,255,255,0.2); }
        @keyframes float { 0%,100% { transform: translateY(0); } 50% { transform: translateY(-8px); } }
        @keyframes thud { 0% { transform: scale(1) rotate(0deg); } 100% { transform: scale(1.05) rotate(-3deg); } }
        @keyframes twitch { 0%,100% { transform: rotate(-2deg); } 50% { transform: rotate(2deg); } }
        @keyframes shimmer { 0% { transform: translateX(-100%); } 100% { transform: translateX(100%); } }
        @keyframes samba { 0%,100% { transform: rotate(0deg); } 25% { transform: rotate(15deg); } 75% { transform: rotate(-15deg); } }
        @keyframes shrink { 0% { transform: scale(1); } 100% { transform: scale(0.6); opacity: 0.5; } }
        @keyframes flop { 0% { transform: rotate(0deg); } 100% { transform: rotate(-90deg) translateY(30px); } }
        @keyframes floatUp { 0% { transform: translateY(0); opacity: 1; } 100% { transform: translateY(-60px); opacity: 0; } }
        @keyframes fight { 0%,100% { transform: translateX(0) scale(1); } 50% { transform: translateX(8px) scale(1.1); } }
        @keyframes clash { 0%,100% { transform: scale(1); } 50% { transform: scale(1.03); } }
        @keyframes flip { 0% { transform: rotateY(0deg); } 100% { transform: rotateY(360deg); } }
        @keyframes shake { 0%,100% { transform: translateX(0); } 20%,60% { transform: translateX(-6px); } 40%,80% { transform: translateX(6px); } }
        @keyframes glitch {
          0%,100% { transform: translate(0); filter: none; }
          20% { transform: translate(-3px, 2px); filter: hue-rotate(90deg); }
          40% { transform: translate(3px, -2px); filter: hue-rotate(-90deg); }
          60% { transform: translate(-2px, 3px); filter: brightness(1.5); }
          80% { transform: translate(2px, -3px); filter: saturate(0); }
        }
        @keyframes riot { 0%,100% { filter: saturate(1); } 50% { filter: saturate(1.5) brightness(1.1); } }
        @keyframes shrinkX { 0% { transform: scaleX(1); } 100% { transform: scaleX(0); } }
        @keyframes marquee { 0% { transform: translateX(100vw); } 100% { transform: translateX(-100%); } }
        @keyframes confetti {
          0% { transform: translateY(100vh) rotate(0deg) scale(0); opacity: 1; }
          50% { transform: translateY(20vh) rotate(180deg) scale(1.5); opacity: 1; }
          100% { transform: translateY(120vh) rotate(360deg) scale(1); opacity: 0; }
        }
      `}</style>
    </div>
  );
}
