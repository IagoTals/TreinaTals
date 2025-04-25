<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Plataforma de Avaliação Anônima de Imagens</title>

<!-- Tailwind CSS e FontAwesome -->
<link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
<link href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.5.2/css/all.min.css" rel="stylesheet">
<link href="https://fonts.googleapis.com/css?family=Roboto:400,700&display=swap" rel="stylesheet">

<!-- Chart.js para o gráfico de ranking -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.3/dist/chart.umd.min.js"></script>

<style>
body         { font-family: 'Roboto', Arial, sans-serif; }
.dropzone    { border: 2px dashed #94a3b8; border-radius: 0.5rem; background:#f8fafc; transition:border-color .2s;}
.dropzone.dragover{border-color:#7c3aed;background:#ede9fe;}
.image-thumb { width:100%; max-width:760px; height:auto; max-height:60vh; object-fit:contain; border-radius:.9rem; box-shadow:0 2px 12px rgba(86,87,205,.13);}
@media (max-width:640px){.image-thumb{max-width:100vw;max-height:74vw;}}
.code-badge  { font-size:1.15em; font-weight:700; background:#ede9fe; color:#5b21b6; padding:.3em .7em; border-radius:.4em; margin-right:.5em; display:inline-block;}
.star        { color:#cbd5e1; font-size:2.2em; cursor:pointer; transition:color .18s; margin:0 3px; display:inline-block; vertical-align:middle;}
.star.selected,.star.filled{color:#f59e42;}
.star.disabled{color:#d1d5db; cursor:not-allowed; pointer-events:none;}
.slider-label{min-width:30px;text-align:center;font-weight:600;}
.fa-star     { filter:drop-shadow(0 0 2px #f5e0ae60);}
</style>
</head>

<body class="bg-gray-50 min-h-screen text-gray-900">
<div class="max-w-3xl mx-auto py-8 px-2 sm:px-4">

<h1 class="text-3xl font-bold mb-3 text-center">Plataforma de Avaliação Anônima de Imagens</h1>
<p class="text-lg text-gray-700 mb-6 text-center">Upload, avalie e descubra quem criou cada imagem, de modo totalmente anônimo.</p>

<!-- Abas -->
<div class="bg-white rounded-xl shadow-md p-4 tab-container">
  <div class="mb-6 flex justify-center space-x-3">
    <button id="tabHost"         class="focus:outline-none px-4 py-2 rounded-t bg-indigo-100 text-indigo-700 font-semibold" onclick="showTab('host')"><i class="fas fa-upload mr-1"></i> Criação de Sessão</button>
    <button id="tabParticipante" class="focus:outline-none px-4 py-2 rounded-t text-gray-700 font-semibold"             onclick="showTab('participante')"><i class="fas fa-users mr-1"></i> Avaliação</button>
    <button id="tabResultados"   class="focus:outline-none px-4 py-2 rounded-t text-gray-700 font-semibold"             onclick="showTab('resultados')"><i class="fas fa-chart-bar mr-1"></i> Resultados</button>
  </div>

  <!-- ================== CRIAÇÃO DE SESSÃO (HOST) ================== -->
  <div id="hostTab">
    <div class="flex items-center mb-4"><div class="w-7 h-7 bg-indigo-200 text-indigo-800 font-bold flex items-center justify-center rounded-full mr-2">1</div><span class="font-medium text-indigo-800">Upload de Imagens</span></div>

    <form id="uploadForm" class="mb-6">
      <label class="block mb-2 text-gray-700 font-medium">Arraste até 30 imagens ou clique para selecionar (JPG/PNG)</label>
      <div id="dropzone" class="dropzone flex flex-col items-center justify-center py-8 mb-4 cursor-pointer">
        <i class="fas fa-image fa-2x mb-2 text-indigo-400"></i>
        <span class="block text-gray-500">Arraste aqui, ou <span class="underline text-indigo-600">clique para selecionar</span></span>
        <input type="file" id="imageInput" accept="image/*" multiple class="hidden" />
      </div>
      <div id="imageList" class="flex flex-wrap gap-3"></div>
      <button type="submit" id="hostCriarSessao" disabled class="mt-6 px-5 py-2 bg-indigo-600 text-white rounded font-bold hover:bg-indigo-700 transition disabled:opacity-50">
        <i class="fas fa-bolt mr-1"></i>Criar Sessão
      </button>
    </form>

    <!-- Sessão criada -->
    <div id="hostSessaoCriada" class="hidden">
      <div class="mb-4">
        <span class="font-medium">Código da Sessão:</span>
        <span id="codigoSessao" class="bg-indigo-100 px-3 py-1 rounded text-indigo-700 font-bold tracking-widest ml-2"></span>
        <button onclick="copiarCodigoSessao()" title="Copiar código" class="ml-2 px-2 py-1 rounded bg-gray-200 hover:bg-gray-300 text-gray-700"><i class="fas fa-copy"></i></button>
      </div>
      <div>
        <h3 class="font-bold mb-2 text-indigo-700">Imagens processadas:</h3>
        <ul id="hostImageMap" class="list-disc pl-5"></ul>
      </div>
      <div class="mt-6 text-center">
        <span class="text-gray-600">Compartilhe o código acima com os participantes.</span><br>
        <button onclick="showTab('participante')" class="mt-4 px-4 py-2 bg-green-600 text-white font-bold rounded hover:bg-green-700 transition">
          Ir para Avaliação <i class="fas fa-arrow-right ml-1"></i>
        </button>
      </div>
    </div>
  </div>

  <!-- ================== AVALIAÇÃO (PARTICIPANTE) ================== -->
  <div id="participanteTab" class="hidden">
    <div class="flex items-center mb-4"><div class="w-7 h-7 bg-indigo-200 text-indigo-800 font-bold flex items-center justify-center rounded-full mr-2">2</div><span class="font-medium text-indigo-800">Avaliação das Imagens</span></div>

    <form id="participanteCodeForm" class="mb-6">
      <label class="block mb-2 font-medium">Digite o código da sessão:</label>
      <input id="codigoParticipante" maxlength="8" class="border border-gray-300 rounded px-4 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-300 w-40" required />
      <button class="ml-2 px-4 py-2 bg-indigo-600 text-white rounded font-semibold hover:bg-indigo-700" type="submit"><i class="fas fa-sign-in-alt mr-1"></i>Entrar</button>
    </form>

    <div id="avaliacaoImagens" class="hidden">
      <div id="avaliacaoGrade" class="flex flex-col gap-8 mb-6"></div>
      <button id="enviarAvaliacoes" disabled class="mt-4 bg-green-600 text-white px-6 py-2 rounded font-bold hover:bg-green-700 transition disabled:opacity-60">
        <i class="fas fa-paper-plane mr-1"></i>Enviar Avaliações
      </button>
      <div id="avaliacaoEnviada" class="mt-4 text-green-700 font-semibold hidden">Avaliações enviadas com sucesso! Obrigado por participar.</div>
    </div>
  </div>

  <!-- ================== RESULTADOS ================== -->
  <div id="resultadosTab" class="hidden">
    <div class="flex items-center mb-4"><div class="w-7 h-7 bg-indigo-200 text-indigo-800 font-bold flex items-center justify-center rounded-full mr-2">3</div><span class="font-medium text-indigo-800">Resultados &amp; Autores</span></div>

    <p class="mb-2 text-gray-600">Use este módulo após as avaliações para descobrir o ranking e os autores.</p>
    <button id="btnVerResultados" onclick="simularResultados()" class="mb-6 px-6 py-2 bg-indigo-600 text-white font-bold rounded hover:bg-indigo-700 transition">
      <i class="fas fa-sync-alt mr-1"></i>Calcular Ranking
    </button>

    <div id="rankingContainer" class="hidden">
      <h3 class="font-bold text-2xl mb-2 text-indigo-700">Ranking das Imagens</h3>
      <canvas id="rankingChart" height="170" class="mb-6"></canvas>
      <div id="rankingImagensGrid" class="flex flex-col gap-6 sm:grid sm:grid-cols-2"></div>

      <button id="btnRevelarAutores" disabled onclick="revelarAutores()" class="mt-4 px-4 py-2 bg-yellow-600 text-white font-bold rounded hover:bg-yellow-700 transition">
        <i class="fas fa-user-secret mr-1"></i>Revelar Autores
      </button>

      <div id="autoresRevelados" class="mt-6 hidden">
        <h4 class="font-bold text-lg mb-2 text-yellow-700"><i class="fas fa-crown mr-1"></i>Autores Revelados</h4>
        <table class="w-full table-auto border rounded shadow">
          <thead><tr class="bg-gray-100"><th class="px-2 py-1 text-left">Imagem</th><th class="px-2 py-1 text-left">Código</th><th class="px-2 py-1 text-left">Autor</th></tr></thead>
          <tbody id="autoresTableBody"></tbody>
        </table>
      </div>
    </div>
  </div>
</div>

<footer class="mt-12 text-gray-400 text-center text-sm">Protótipo para avaliações anônimas de imagens &middot; 2024</footer>
</div>

<!-- ================== SCRIPT ================== -->
<script>
/* ----- Navegação de Abas ----- */
function capitalize(s){return s.charAt(0).toUpperCase()+s.slice(1);}
function showTab(tab){['host','participante','resultados'].forEach(t=>{
 document.getElementById(t+'Tab').classList.toggle('hidden',t!==tab);
 document.getElementById('tab'+capitalize(t)).classList.toggle('bg-indigo-100',t===tab);
 document.getElementById('tab'+capitalize(t)).classList.toggle('text-indigo-700',t===tab);
 document.getElementById('tab'+capitalize(t)).classList.toggle('text-gray-700',t!==tab);
});}
showTab('host');

/* ----- Estados ----- */
const sessionState={codigo:'',images:[],imageCodes:[],imageAuthorMap:{},notasPorImagem:{},participantes:[]};
let avaliacaoOrdemAleatoria=[];

/* =========== HOST ============ */
const dropzone=document.getElementById('dropzone');
const imageInput=document.getElementById('imageInput');
const imageList=document.getElementById('imageList');
const uploadForm=document.getElementById('uploadForm');
const hostCriarSessao=document.getElementById('hostCriarSessao');
let arquivosSelecionados=[];

dropzone.onclick=()=>imageInput.click();
dropzone.ondragover=e=>{e.preventDefault();dropzone.classList.add('dragover');};
dropzone.ondragleave=()=>dropzone.classList.remove('dragover');
dropzone.ondrop=e=>{e.preventDefault();dropzone.classList.remove('dragover');handleFiles(e.dataTransfer.files);};
imageInput.onchange=e=>handleFiles(e.target.files);

function handleFiles(fileList){
 arquivosSelecionados=Array.from(fileList).filter(f=>f.type.startsWith('image/')).slice(0,30);
 renderImageThumbs(arquivosSelecionados);
}

function renderImageThumbs(files){
 imageList.innerHTML='';
 files.forEach((file,idx)=>{
   const code=String.fromCharCode(65+idx);
   const url=URL.createObjectURL(file);
   const wrap=document.createElement('div');wrap.className='flex flex-col items-center';
   const img=document.createElement('img');img.src=url;img.className='image-thumb mb-1';img.alt='Imagem '+code;
   const lbl=document.createElement('span');lbl.className='code-badge';lbl.textContent=code;
   wrap.append(img,lbl);imageList.appendChild(wrap);
 });
 hostCriarSessao.disabled=!(files.length>=2&&files.length<=30);
}

uploadForm.onsubmit=async e=>{
 e.preventDefault();
 if(arquivosSelecionados.length<2||arquivosSelecionados.length>30)return;

 const autores=[];
 for(let i=0;i<arquivosSelecionados.length;i++){
   const def='Autor '+String.fromCharCode(65+i);
   autores.push(prompt('Nome de quem enviou a Imagem '+String.fromCharCode(65+i),def)||def);
 }

 sessionState.codigo=gerarCodigoSessao();
 sessionState.images=[];
 sessionState.imageCodes=[];
 sessionState.imageAuthorMap={};

 arquivosSelecionados.forEach((file,i)=>{
   const code=String.fromCharCode(65+i);
   sessionState.images.push({code,file,url:URL.createObjectURL(file),nomeAutor:autores[i]});
   sessionState.imageCodes.push(code);
   sessionState.imageAuthorMap[code]=autores[i];
 });
 mostrarSessaoCriada();
};

function gerarCodigoSessao(){const chars='ABCDEFGHJKMNPQRSTUVWXYZ23456789';let c='';for(let i=0;i<6;i++)c+=chars[Math.floor(Math.random()*chars.length)];return c;}

function mostrarSessaoCriada(){
 uploadForm.classList.add('hidden');
 document.getElementById('hostSessaoCriada').classList.remove('hidden');
 document.getElementById('codigoSessao').textContent=sessionState.codigo;
 const ul=document.getElementById('hostImageMap');ul.innerHTML='';
 sessionState.images.forEach(img=>{
   const li=document.createElement('li');
   li.innerHTML=`<span class="code-badge">${img.code}</span> <span class="text-gray-700">${img.nomeAutor}</span>`;
   ul.appendChild(li);
 });
}
function copiarCodigoSessao(){navigator.clipboard.writeText(sessionState.codigo);alert('Código copiado!');}

/* =========== PARTICIPANTE ============ */
const participanteCodeForm=document.getElementById('participanteCodeForm');
const codigoParticipanteInput=document.getElementById('codigoParticipante');
const avaliacaoImagens=document.getElementById('avaliacaoImagens');
const avaliacaoGrade=document.getElementById('avaliacaoGrade');
const enviarAvaliacoesBtn=document.getElementById('enviarAvaliacoes');
const avaliacaoEnviadaMsg=document.getElementById('avaliacaoEnviada');
let participanteAvaliacoes={};

participanteCodeForm.onsubmit=e=>{
 e.preventDefault();
 if(codigoParticipanteInput.value.trim().toUpperCase()!==sessionState.codigo){alert('Código inválido.');return;}
 participanteCodeForm.classList.add('hidden');
 iniciarAvaliacao();
};

function embaralhar(a){for(let i=a.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[a[i],a[j]]=[a[j],a[i]];}return a;}
function criarEstrelas(imgCode,onChange){
 const cont=document.createElement('div');cont.className='flex items-center gap-1 justify-center';
 const val=document.createElement('span');val.className='ml-2 slider-label text-yellow-700 text-lg font-bold';cont.appendChild(val);
 let current=1;participanteAvaliacoes[imgCode]=1;
 function render(v){Array.from(cont.querySelectorAll('.star')).forEach((st,i)=>st.classList.toggle('selected',v>i));val.textContent=v;}
 for(let i=1;i<=5;i++){
   const st=document.createElement('i');st.className='star fas fa-star';st.onclick=()=>{current=i;participanteAvaliacoes[imgCode]=i;render(i);onChange(i);};
   st.onkeydown=e=>{if((e.key==='Enter'||e.key===' ')&&!st.classList.contains('disabled')){st.click();e.preventDefault();}};
   cont.insertBefore(st,val);
 }
 const noteMin=document.createElement('span');noteMin.className='text-xs text-gray-600 ml-2';noteMin.textContent='(mín: 1)';
 cont.append(noteMin);render(1);return cont;
}

function iniciarAvaliacao(){
 avaliacaoImagens.classList.remove('hidden');
 avaliacaoGrade.innerHTML='';participanteAvaliacoes={};
 avaliacaoOrdemAleatoria=embaralhar(sessionState.images);
 avaliacaoOrdemAleatoria.forEach(img=>{
   const card=document.createElement('div');card.className='bg-gray-100 p-5 rounded shadow flex flex-col items-center';
   card.innerHTML=`<img src="${img.url}" alt="Imagem ${img.code}" class="image-thumb mb-2"><span class="font-semibold mb-1 text-indigo-700">Imagem ${img.code}</span>`;
   const stars=criarEstrelas(img.code,validarEnvio);card.appendChild(stars);
   avaliacaoGrade.appendChild(card);
 });
 enviarAvaliacoesBtn.disabled=false;avaliacaoEnviadaMsg.classList.add('hidden');validarEnvio();
}

function validarEnvio(){enviarAvaliacoesBtn.disabled=sessionState.imageCodes.some(c=>!participanteAvaliacoes[c]);}
enviarAvaliacoesBtn.onclick=()=>{
 const nome=prompt('Digite um apelido (opcional):','Participante '+Math.ceil(Math.random()*100))||'Participante';
 sessionState.participantes.push(nome);
 sessionState.imageCodes.forEach(c=>{
   sessionState.notasPorImagem[c]=sessionState.notasPorImagem[c]||[];
   sessionState.notasPorImagem[c].push({nota:participanteAvaliacoes[c],participante:nome});
 });
 avaliacaoEnviadaMsg.classList.remove('hidden');enviarAvaliacoesBtn.disabled=true;
 avaliacaoGrade.querySelectorAll('.star').forEach(st=>st.classList.add('disabled'));
};

/* =========== RESULTADOS ============ */
let chart=null;
function simularResultados(){
 if(!sessionState.images.length){alert('Nenhuma sessão ativa.');return;}
 const ranking=sessionState.images.map(img=>{
   const notas=(sessionState.notasPorImagem[img.code]||[]).map(o=>o.nota);
   const media=notas.length?notas.reduce((a,b)=>a+b,0)/notas.length:0;
   return {...img,media,votos:notas.length};
 }).sort((a,b)=>b.media-a.media);
 mostrarRanking(ranking);mostrarRankingImagens(ranking);window.rankingArr=ranking;
}
function mostrarRanking(ranking){
 document.getElementById('rankingContainer').classList.remove('hidden');
 document.getElementById('btnVerResultados').disabled=true;
 const ctx=document.getElementById('rankingChart').getContext('2d');
 if(chart)chart.destroy();
 chart=new Chart(ctx,{type:'bar',data:{labels:ranking.map(r=>'Imagem '+r.code),datasets:[{data:ranking.map(r=>+r.media.toFixed(2)),backgroundColor:'#f59e42',borderRadius:9}]},
   options:{indexAxis:'y',plugins:{legend:{display:false}},scales:{x:{min:0,max:5,beginAtZero:true}}}});
 document.getElementById('btnRevelarAutores').disabled=false;
}
function mostrarRankingImagens(ranking){
 const grid=document.getElementById('rankingImagensGrid');grid.innerHTML='';
 ranking.forEach(r=>{
  const d=document.createElement('div');d.className='flex flex-col items-center bg-gray-100 rounded-lg p-3 shadow relative';
  d.innerHTML=`<span class="absolute top-1 left-2 bg-indigo-100 text-indigo-700 px-2 py-0.5 rounded text-sm font-bold">${r.code}</span>
               <img src="${r.url}" class="image-thumb mb-2" alt="Imagem ${r.code}">
               <span class="font-bold text-yellow-700 mb-1">Nota média: ${r.media?r.media.toFixed(2):'-'}</span>
               <span class="text-sm text-gray-600">Votos: ${r.votos}</span>`;
  grid.appendChild(d);
 });
}
function revelarAutores(){
 document.getElementById('autoresRevelados').classList.remove('hidden');
 document.getElementById('btnRevelarAutores').disabled=true;
 const tbody=document.getElementById('autoresTableBody');tbody.innerHTML='';
 window.rankingArr.forEach(img=>{
   const tr=document.createElement('tr');
   tr.innerHTML=`<td class="border-b px-2 py-1"><img src="${img.url}" class="rounded" style="max-width:72px;max-height:52px" alt=""></td>
                 <td class="border-b px-2 py-1"><span class="code-badge">${img.code}</span></td>
                 <td class="border-b px-2 py-1 text-indigo-800">${img.nomeAutor}</td>`;
   tbody.appendChild(tr);
 });
}
</script>
</body>
</html>
