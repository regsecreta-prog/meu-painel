<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>
  <title>Painel Rede Executora - Regulacao RO</title>

  <!-- Leaflet -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

  <style>
    :root{
      --bg:#0b1220;
      --panel:#0f1b33;
      --ink:#e8eefc;
      --muted:#a9b6d3;
      --line:#223052;
      --accent:#2b6cff;
      --chip:#122145;
      --shadow: 0 10px 30px rgba(0,0,0,.35);
      --hot:#ff2d2d;
    }

    *{box-sizing:border-box}
    body{margin:0;font-family:system-ui, Segoe UI, Roboto, Arial;background:var(--bg);color:var(--ink);}

    header{
      padding:14px 18px;border-bottom:1px solid var(--line);
      display:flex;justify-content:space-between;gap:12px;align-items:flex-end;
      background:linear-gradient(180deg, rgba(255,255,255,.03), rgba(255,255,255,0));
    }
    header h1{margin:0;font-size:16px;letter-spacing:.2px}
    header small{color:var(--muted)}

    /* Texto do município (hover/click) */
    #mapHover{
      margin-top:8px;
      font-size:16px;
      font-weight:800;
      max-width:70vw;
      white-space:normal;
      line-height:1.2;
    }
    .hoverDefault{ color: var(--muted); font-weight:600; }
    .hoverHover{ color: var(--ink); font-weight:900; font-size:17px; }
    .hoverStrong{ color: var(--hot); font-weight:950; font-size:17px; letter-spacing:.2px; }

    /* layout */
    .wrap{
      display:grid;
      grid-template-columns: 0.8fr 2.2fr;
      gap:12px;
      padding:12px;
      height:calc(100vh - 64px);
    }
    #map{border:1px solid var(--line);border-radius:14px;overflow:hidden;box-shadow:var(--shadow);}

    /* painel direito (filtros fixos + scroll só embaixo) */
    .panel{
      border:1px solid var(--line);
      border-radius:14px;
      background:var(--panel);
      padding:14px;
      box-shadow:var(--shadow);

      overflow:hidden;
      display:flex;
      flex-direction:column;
      height:calc(100vh - 64px);
    }
    .stickyFilters{
      position:sticky;
      top:0;
      z-index:5;
      background:var(--panel);
      padding-bottom:10px;
      border-bottom:1px solid var(--line);
    }
    .scrollArea{
      overflow:auto;
      padding-top:10px;
    }

    .titleRow{display:flex;justify-content:space-between;align-items:flex-start;gap:12px;margin-bottom:10px;}
    .panel h2{margin:0;font-size:16px;letter-spacing:.2px}
    .meta{color:var(--muted);font-size:12.5px;margin:6px 0 12px;}

    .controls{display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:10px;}
    .control{border:1px solid var(--line);border-radius:12px;padding:10px;background:rgba(255,255,255,.02);}
    .control label{display:block;color:var(--muted);font-size:12px;margin-bottom:6px}

    .btn,.input{
      width:100%;
      padding:9px 10px;
      border-radius:10px;
      border:1px solid var(--line);
      background:#0b1220;
      color:var(--ink);
      outline:none;
      font-size:13px;
    }
    .btn:hover,.input:hover{border-color:#35508b}
    .btn:focus,.input:focus{border-color:#35508b;box-shadow:0 0 0 2px rgba(43,108,255,.25);}
    select.btn option{background:#0b1220;color:var(--ink);}

    .chip{
      display:inline-block;
      padding:3px 10px;
      border-radius:999px;
      background:var(--chip);
      border:1px solid var(--line);
      color:var(--muted);
      font-size:12px;
      white-space:nowrap;
    }

    /* toolbar clean */
    .toolbar{display:flex;justify-content:flex-end;gap:10px;margin:8px 0 0;}
    .toolBtn{
      padding:10px 12px;border-radius:12px;border:1px solid var(--line);
      background:rgba(255,255,255,.02);cursor:pointer;user-select:none;
      display:flex;align-items:center;gap:8px;
    }
    .toolBtn:hover{border-color:#35508b}

    .box{
      border:1px solid var(--line);
      border-radius:14px;
      background:rgba(255,255,255,.03);
      padding:12px;
      margin-top:10px;
    }
    .box h3{margin:0 0 6px;font-size:13px;color:#cfe0ff;letter-spacing:.2px;}
    .muted{color:var(--muted);font-size:12px}
    .hidden{display:none !important;}

    /* Resultado com cara de relatório (tabela) */
    .tableWrap{
      border:1px solid var(--line);
      border-radius:12px;
      overflow:hidden;
      background:rgba(255,255,255,.02);
    }
    table{width:100%;border-collapse:separate;border-spacing:0;font-size:13px;}
    thead th{
      text-align:left;padding:10px;color:#cfe0ff;
      background:rgba(255,255,255,.03);
      border-bottom:1px solid var(--line);
      font-weight:700;letter-spacing:.2px;
      position:sticky; top:0; z-index:1;
    }
    tbody td{padding:10px;border-bottom:1px solid rgba(34,48,82,.6);vertical-align:top;}
    tbody tr:hover td{ background:rgba(43,108,255,.06); }
    tbody tr:last-child td{ border-bottom:none; }

    .badge{
      display:inline-flex;align-items:center;gap:6px;
      padding:4px 10px;border-radius:999px;
      border:1px solid var(--line);
      background:rgba(18,33,69,.9);
      color:var(--muted);
      font-size:12px;
      white-space:nowrap;
    }
    .badge strong{color:var(--ink);font-weight:800}

    a.linkBtn, button.linkBtn{
      display:inline-block;
      padding:7px 10px;
      border-radius:10px;
      border:1px solid var(--line);
      background:#0b1220;
      color:var(--ink);
      text-decoration:none;
      white-space:nowrap;
      cursor:pointer;
      font-family:inherit;
      appearance:none;
      font-size:13px;
    }
    a.linkBtn:hover, button.linkBtn:hover{border-color:#35508b}
    button.linkBtn:disabled{opacity:.55; cursor:not-allowed;}

    /* Painel destacado (SERVIÇOS) */
    .procGrid{display:grid;grid-template-rows: 320px 1fr;gap:10px;}
    .procListArea{border:1px solid var(--line);border-radius:14px;background:rgba(255,255,255,.02);padding:10px;overflow:hidden;}
    .procListHeader{display:flex;gap:10px;align-items:center;margin-bottom:10px;}
    .procList{height:250px;overflow:auto;padding-right:6px;}
    .procItem{display:flex;justify-content:space-between;align-items:flex-start;gap:10px;padding:10px;border:1px solid var(--line);border-radius:12px;background:rgba(255,255,255,.02);margin-bottom:8px;cursor:pointer;}
    .procItem:hover{ border-color:#35508b; }
    .procItem b{ font-size:13px; }
    .procItem small{ color:var(--muted); display:block; margin-top:2px; }
    .procItem.active{border-color: var(--accent);box-shadow: 0 0 0 2px rgba(43,108,255,.18);}
    .procDetailArea{border:1px solid var(--line);border-radius:14px;background:rgba(255,255,255,.02);padding:10px;min-height:220px;}

    /* Rodapé dentro do painel (área rolável) */
    .devFooterPanel{
      margin-top: 18px;
      padding: 14px;
      border-radius: 14px;
      border: 1px solid var(--line);
      background: rgba(255,255,255,.03);
      text-align: center;
      color: var(--muted);
      font-size: 13px;
    }
    .devFooterPanel b{ color: var(--ink); font-weight: 900; }

    /* ===============================
       AUTOCOMPLETE PROFISSIONAL
       =============================== */
    .acWrap{ position:relative; }
    #procInput{
      font-size: 12.5px;   /* ✅ letra menor no input */
      padding: 8px 10px;
    }
    .acList{
      position:absolute;
      left:0;
      right:0;             /* ocupa toda a largura do controle */
      top: calc(100% + 8px);
      z-index: 9999;
      border:1px solid var(--line);
      border-radius: 14px;
      background: rgba(11,18,32,.98);
      box-shadow: var(--shadow);
      max-height: 320px;
      overflow:auto;
      padding:8px;
      display:none;
    }
    .acList.show{ display:block; }

    .acHeader{
      display:flex;
      justify-content:space-between;
      align-items:center;
      gap:10px;
      padding:6px 8px 10px;
      border-bottom:1px solid rgba(34,48,82,.6);
      margin-bottom:8px;
    }
    .acHeader .chip{ font-size:11.5px; }

    .acItem{
      display:flex;
      justify-content:space-between;
      gap:10px;
      padding:10px 10px;
      border:1px solid rgba(34,48,82,.7);
      border-radius:12px;
      background: rgba(255,255,255,.02);
      cursor:pointer;
      margin-bottom:8px;
    }
    .acItem:hover{ border-color:#35508b; background: rgba(43,108,255,.06); }
    .acItem.active{ border-color: var(--accent); box-shadow: 0 0 0 2px rgba(43,108,255,.18); }

    .acMain{
      min-width:0;
    }
    .acMain b{
      display:block;
      font-size:12.5px;    /* ✅ letra menor na lista */
      line-height:1.2;
      word-break:break-word;
    }
    .acMain small{
      display:block;
      color: var(--muted);
      font-size:11.5px;    /* ✅ menor ainda */
      margin-top:3px;
      line-height:1.2;
    }
    .acTag{
      flex:0 0 auto;
      align-self:flex-start;
      display:inline-flex;
      padding:4px 8px;
      border-radius:999px;
      border:1px solid rgba(34,48,82,.7);
      background: rgba(18,33,69,.75);
      color: var(--muted);
      font-size:11px;
      white-space:nowrap;
    }
    .acMark{
      color:#cfe0ff;
      font-weight:900;
      text-decoration: underline;
      text-underline-offset: 2px;
    }

    @media (max-width: 980px){
      .wrap{grid-template-columns:1fr; height:auto;}
      #map{height:46vh;}
      .panel{height:auto;}
      .procGrid{ grid-template-rows: 260px 1fr; }
      .procList{ height:200px; }
      .acList{ max-height: 260px; }
    }
  </style>
</head>

<body>
<header>
  <div>
    <h1> COORDENADORIA DE REGULAÇÃO DE ACESSO AOS SERVIÇOES DE SAÚDE - CREG/RO -  PAINEL REDE EXECUTORA </h1>
    <small>Clique no municipio no mapa ou use filtros (Servicos / Procedimentos)</small>
    <div id="mapHover" class="hoverDefault">Passe o mouse no mapa para ver o município aqui.</div>
  </div>
  <small id="status">Carregando malha...</small>
</header>

<div class="wrap">
  <div id="map"></div>

  <div class="panel">
    <!-- FIXO -->
    <div class="stickyFilters">
      <div class="titleRow">
        <div>
          <h2>Filtros</h2>
          <div class="meta" id="meta">Selecione um municipio para visualizar a rede ofertada.</div>
        </div>
        <div class="chip" id="chip">Sem filtro</div>
      </div>

      <div class="controls">
        <div class="control">
          <label for="modeSel">Modo</label>
          <select id="modeSel" class="btn">
            <option value="servicos">SERVICOS (Rede por municipio)</option>
            <option value="procedimentos">PROCEDIMENTOS (Rol por unidade)</option>
          </select>
        </div>

        <!-- SERVICOS: MUNICIPIO -->
        <div class="control" id="munControl">
          <label for="munSel">Municipio</label>
          <select id="munSel" class="btn">
            <option value="">Todos</option>
          </select>
        </div>

        <!-- SERVICOS: TIPO -->
        <div class="control" id="tipoProcControl">
          <label for="tipoProcSel">Tipo de Procedimento</label>
          <select id="tipoProcSel" class="btn">
            <option value="">Todos</option>
          </select>
        </div>

        <!-- PROCEDIMENTOS: UNIDADE -->
        <div class="control hidden" id="unitControl">
          <label for="unitSel">Unidade (Prestador)</label>
          <select id="unitSel" class="btn">
            <option value="">Todas</option>
          </select>
        </div>

        <!-- PROCEDIMENTOS: PERFIL -->
        <div class="control hidden" id="perfilControl">
          <label for="perfilSel">Perfil</label>
          <select id="perfilSel" class="btn">
            <option value="">Ambulatorial e Hospitalar</option>
            <option value="AMBULATORIAL">Ambulatorial</option>
            <option value="HOSPITALAR">Hospitalar</option>
          </select>
        </div>

        <!-- PROCEDIMENTOS: autocomplete custom (profissional) -->
        <div class="control hidden" id="procControl" style="grid-column:1 / span 2;">
          <label for="procInput">Procedimento (pesquise e selecione)</label>

          <div class="acWrap">
            <input id="procInput" class="input" placeholder="Ex.: tomo, hernio, prosta..." autocomplete="off" />
            <div id="procSuggest" class="acList" role="listbox" aria-label="Sugestões de procedimentos"></div>
          </div>

          <div class="muted" style="margin-top:8px;">
            Dica: Enter seleciona o item destacado • Esc fecha • ↑ ↓ navega.
          </div>
        </div>
      </div>

      <div class="toolbar">
        <div class="toolBtn" id="btnLimpar" title="Limpa todos os filtros e volta ao mapa inicial">
          Limpar filtros
        </div>
      </div>
    </div>

    <!-- ROLÁVEL -->
    <div class="scrollArea">
      <div class="box">
        <h3 id="boxTitle">Resultado</h3>
        <div class="meta" id="boxMeta">Clique no municipio no mapa para ver a rede ofertada.</div>
        <div id="results"></div>
      </div>

      <!-- Painel destacado: Procedimentos no modo SERVIÇOS -->
      <div class="box hidden" id="procPanel">
        <h3 id="procPanelTitle">Procedimentos (Lista)</h3>
        <div class="meta" id="procPanelMeta">Clique em “Procedimentos” em uma unidade no Resultado para listar aqui.</div>

        <div class="procGrid">
          <div class="procListArea">
            <div class="procListHeader">
              <div class="chip" id="procCountChip">0 itens</div>
              <input id="procListFilter" class="input" placeholder="Filtrar na lista (ex.: hernio, prosta, tomo...)" />
            </div>
            <div id="procList" class="procList"></div>
          </div>

          <div class="procDetailArea">
            <div class="procItem" style="margin-bottom:0; cursor:default;">
              <div>
                <b id="procDetailTitle">Detalhe</b>
                <small id="procDetailMeta">Selecione um procedimento na lista acima.</small>
              </div>
              <div><span class="chip" id="procDetailChip">—</span></div>
            </div>
            <div id="procDetailBody" style="margin-top:10px;"></div>
          </div>
        </div>
      </div>

      <div class="box">
        <h3 id="detTitle">Detalhe</h3>
        <div class="meta" id="detMeta">Selecione uma unidade no modo Procedimentos, ou clique no municipio no mapa.</div>
        <div id="detail"></div>
      </div>

      <!-- Rodapé dentro da área rolável -->
      <div class="devFooterPanel">
        Desenvolvido por <b>Renato Castro</b> • Versão 1 / 2026
      </div>
    </div>
  </div>
</div>

<script>
/* ============================================================
   BASE - SERVICOS (rede)
   ============================================================ */
const rede = {
  "PORTO VELHO": [
    { servico: "CIRURGIAS (CIRURGIA GERAL)", prestador: "HOSPITAL SAMAR - PORTO VELHO", link: "https://drive.google.com/file/d/1QKUEnbgspA2ht1ByXny9tiqCJu1ugz6Y/view?usp=sharing" },
    { servico: "CIRURGIAS (ORTOPEDIA E CIRURGIA GERAL)", prestador: "HOSPITAL 9 DE JULHO DE RONDONIA LTDA", link: "https://drive.google.com/file/d/1epviiRQPUTLL2AsiL2IT0hsMH3Euew0M/view?usp=sharing" },
    { servico: "CIRURGIAS (ORTOPEDIA E CIRURGIA GERAL)", prestador: "GATE-SERVICOS MEDICO HOSPITALARES LTDA" },
    { servico: "CIRURGIAS (ORTOPEDIA, CIRURGIA GERAL E URO)", prestador: "CASA DE SAUDE SANTA MARCELINA" },
    { servico: "CIRURGIAS (ORTOPEDIA, UROLOGIA)", prestador: "HOSP-COR HOSPITAL DO CORACAO DE RONDONIA (PRONTOCORDIS)" },
    { servico: "CIRURGIAS (ORTOPEDIA, CIRURGIA GERAL E URO)", prestador: "IRB PRIME CARE SERVICOS MEDICOS CLINICO HOSPITALAR LTDA" },
    { servico: "CIRURGIAS (ORTOPEDIA, CIRURGIA GERAL E URO)", prestador: "INSTITUTO DO CORACAO DE RONDONIA", link: "https://drive.google.com/file/d/1jvCO6TJ7GyybErP24GrIpPtKonO2hL5m/view?usp=drive_link" },
    { servico: "CIRURGIAS (ORTOPEDIA, CIRURGIA GERAL E URO)", prestador: "HOSPITAL DE BASE DR. ARY PINHEIRO" },

    { servico: "DIAGNOSTICO: RM", prestador: "HOSP-COR HOSPITAL DO CORACAO DE RONDONIA LTDA" },
    { servico: "DIAGNOSTICO: RM", prestador: "ALPHACLIN - UNIDADE DE ULTRASSONOGRAFIA DE RONDONIA LTDA" },
    { servico: "DIAGNOSTICO: RM", prestador: "CENTRO DE DIAGNOSTICO RADIOIMAGEM LTDA" },
    { servico: "DIAGNOSTICO: RM", prestador: "ENOCH - UNIDADE DE RADIODIAGNOSTICO E ULTRA-SONOGRAFIA LTDA" },
   
    { servico: "DIAGNOSTICO: ELETROENCEFALO E ELETRONEURO", prestador: "NEUROPH" },

    { servico: "DIAGNOSTICO / TERAPIA ONCOLOGICA", prestador: "HOSPITAL DE AMOR AMAZONIA" },

    { servico: "DIAGNOSTICO: TC", prestador: "CENTRO DE DIAGNOSTICO RADIOIMAGEM LTDA" },
    { servico: "DIAGNOSTICO: TC", prestador: "HOSPITAL SAMAR S/A - PORTO VELHO" },
    { servico: "DIAGNOSTICO: TC", prestador: "GATE-SERVICOS MEDICO HOSPITALARES LTDA" },
    { servico: "DIAGNOSTICO: TC", prestador: "HOSP-COR HOSPITAL DO CORACAO DE RONDONIA LTDA" },
    { servico: "DIAGNOSTICO (ULTRASSONOGRAFIA)", prestador: "HOSPITAL DE RETAGUARDA DE RONDONIA - HRRO" },


    { servico: "DIAGNOSTICO: CINTILOGRAFIA", prestador: "CENTRO DE MEDICINA NUCLEAR DE RONDONIA LTDA" },
    { servico: "DIAGNOSTICO: DENSITOMETRIA", prestador: "DENSYA MEDICINA ESPECIALIZADA" },

    { servico: "HEMODIALISE AMBULATORIAL (TRS)", prestador: "NEFRON SERVICOS DE NEFROLOGIA LTDA" },
    { servico: "HEMODIALISE AMBULATORIAL (TRS)", prestador: "CLINICA RENAL DE RONDONIA LTDA (CLINERON)" },

    { servico: "HEMODINAMICA", prestador: "ANGIOCENTER" },
    { servico: "HEMODINAMICA", prestador: "CCATE - CENTRO CARDIOLOGICO SOARES E COELHO LTDA" },
    { servico: "HEMODINAMICA", prestador: "NOVECATE" },
    { servico: "HEMODINAMICA", prestador: "NEUROCORDIS" },
    { servico: "HEMODINAMICA", prestador: "EQUILIBRIUM MULTI SERVICOS DE SAUDE LTDA" },

    { servico: "NEUROCIRURGIA", prestador: "IRB PRIME CARE SERVICOS MEDICOS CLINICOS HOSPITALARES LTDA (LOTE II E III)" },
    { servico: "NEUROCIRURGIA", prestador: "INAO SERVICOS MEDICOS LTDA (LOTE I - HBAP/HICD)" },

    { servico: "OFTALMOLOGIA", prestador: "SOL SERVICOS DE OFTALMOLOGIA LTDA (CLINICA SOL)" },

    { servico: "DIAGNOSTICO: ANATOMOPATOLOGICO", prestador: "DIAC - INSTITUTO PAULISTA DE MEDICINA DE PORTO VELHO LTDA" },
    { servico: "DIAGNOSTICO: ANATOMOPATOLOGICO", prestador: "LABORATORIO BIO CHECK UP LTDA" },
    { servico: "DIAGNOSTICO: ANATOMOPATOLOGICO", prestador: "CEPAM - L L S VERAS LTDA" },

    { servico: "TRANSPORTE AEROMEDICO", prestador: "RIMA - RIO MADEIRA AVIACAO LTDA" },
    { servico: "COMUNIDADES TERAPEUTICAS", prestador: "ASSOCIACAO CASA FAMILIA ROSETTA" },
    { servico: "LEITOS CLINICOS", prestador: "HOSPITAL 9 DE JULHO DE RONDONIA LTDA" }
  ],

  "CACOAL": [
    { servico: "CIRURGIAS (ORTOPEDIA)", prestador: "AZEVEDO & AZEVEDO LTDA (HOSPITAL DOS ACIDENTADOS)" },
    { servico: "DIAGNOSTICO: TC E RM", prestador: "ANGA - CENTRO DE MEDICINA DIAGNOSTICA LTDA" },
    { servico: "DIAGNOSTICO: TC", prestador: "NOVA IMAGEM DIAGNOSTICO" },
    { servico: "HEMODINAMICA", prestador: "CARDIO CENTER" },
    { servico: "OFTALMOLOGIA", prestador: "NEGREIROS & VENTORIN SERVICOS MEDICOS LTDA" },
    { servico: "TERAPIA INTENSIVA", prestador: "HOSPITAL SAMAR S/A - CACOAL" },
    { servico: "COMUNIDADES TERAPEUTICAS", prestador: "COMUNIDADE TERAPEUTICA ABISAI" }
  ],

  "JI-PARANÁ": [
    { servico: "CIRURGIAS (ORTOPEDIA, CIRURGIA GERAL E URO)", prestador: "HOSPITAL CANDIDO RONDON (SSY HOLDING LTDA)", link: "https://drive.google.com/file/d/1ev6UCrRkEs8vs-mC9YeLMUG51yN9s8YK/view?usp=drive_link" },
    { servico: "CIRURGIA CARDIACA", prestador: "HOSPITAL CANDIDO RONDON (SSY HOLDING LTDA)" },
    { servico: "DIAGNOSTICO: TC E RM", prestador: "FUNDACAO PIO XII" },
    { servico: "OFTALMOLOGIA", prestador: "INSTITUTO OFTALMOLOGICO DO BRASIL LTDA (SANTA CASA DE JI PARANA)" },
    { servico: "OFTALMOLOGIA", prestador: "SOL SERVICOS DE OFTALMOLOGIA (SOL OFTALMOLOGIA)" },
    { servico: "TERAPIA INTENSIVA", prestador: "HOSPITAL SAMAR S/A - JI PARANA" }
  ],

  "ARIQUEMES": [
    { servico: "DIAGNOSTICO: RM", prestador: "IMEDI" },
    { servico: "OFTALMOLOGIA", prestador: "NEGREIROS & VENTORIN SERVICOS MEDICOS LTDA (MEDICOS DOS OLHOS)" },
    { servico: "TERAPIA INTENSIVA", prestador: "HOSPITAL MONTE SINAI" }
  ],

  "VILHENA": [
    { servico: "CIRURGIAS (ORTOPEDIA)", prestador: "HOSPITAL BOM JESUS LTDA", link: "URL_DA_CARTA_DE_SERVICOS" },
    { servico: "DIAGNOSTICO: TC E RM", prestador: "C D I CLINICA DE RADIOLOGIA E DIAGNOSTICO POR IMAGEM" },
    { servico: "COMUNIDADES TERAPEUTICAS", prestador: "ASSOCIACAO TRINDADE SANTA DE VILHENA" }
  ],

  "ROLIM DE MOURA": [
    { servico: "DIAGNOSTICO: TC E RM", prestador: "CENTRO DE DIAGNOSTICO MULTIMAGEM LTDA" },
    { servico: "COMUNIDADES TERAPEUTICAS", prestador: "COMUNIDADE TERAPEUTICA NOVA ALIANCA - CERNA" }
  ],

  "CEREJEIRAS": [
    { servico: "DIAGNOSTICO: TC", prestador: "MEGA IMAGEM" },
    { servico: "DIAGNOSTICO: TC", prestador: "CLIMEDI LTDA" }
  ],

  "GUAJARA-MIRIM": [
    { servico: "DIAGNOSTICO: TC", prestador: "BOUE E RUIZ LTDA-EPP (CLINI MEDI)" }
  ],

  "JARU": [
    { servico: "DIAGNOSTICO: RM", prestador: "GASTRO IMAGEM LTDA" }
  ],

  "OURO PRETO DO OESTE": [
    { servico: "DIAGNOSTICO: TC", prestador: "HOSPITAL SAO LUCAS DE OURO PRETO LTDA" },
    { servico: "TERAPIA INTENSIVA", prestador: "HOSPITAL SAO LUCAS DE OURO PRETO LTDA" }
  ],

  "PIMENTA BUENO": [
    { servico: "DIAGNOSTICO: ELETROENCEFALO E ELETRONEURO", prestador: "R D SERVICOS MEDICOS LTDA" }
  ]
};

/* ============================================================
   BASE - PROCEDIMENTOS (providers)
   - padronizado: chaves (id, nome, municipio, tipo, perfil, link, procedimentos)
   - procedimentos sem acentos
   - correcoes: PERFIL -> perfil, duplicidade de "perfil", nome com parenteses a mais
   ============================================================ */
const providers = [
  /* --- CIRURGIAS (exemplos com rol) --- */
  {
  id: "HBJ_VILHENA",
  nome: "HOSPITAL BOM JESUS LTDA",
  municipio: "VILHENA",
  tipo: "CIRURGIAS (ORTOPEDIA/UROLOGIA/CLINICA CIRURGICA)",
  perfil: "Hospitalar",
  procedimentos: [
    "CISTOSTOMIA",
    "INSTALACAO ENDOSCOPICA DE CATETER DUPLO J",
    "NEFRECTOMIA PARCIAL",
    "NEFRECTOMIA TOTAL",
    "NEFROURETERECTOMIA TOTAL",
    "RESSECCAO DO COLO VESICAL / TUMOR VESICAL A CEU ABERTO",
    "TRATAMENTO CIRURGICO DE HEMORRAGIA VESICAL (FORMOLIZACAO DA BEXIGA)",
    "URETERECTOMIA",
    "URETEROLITOTOMIA",
    "MEATOTOMIA SIMPLES",
    "URETROTOMIA INTERNA",
    "PROSTATECTOMIA SUPRAPUBICA",
    "RESSECCAO ENDOSCOPICA DE PROSTATA",
    "ORQUIDOPEXIA UNILATERAL",
    "ORQUIECTOMIA UNILATERAL",
    "EXERESE DE CISTO VAGINAL",
    "ARTRODESE CERVICAL ANTERIOR TRES NIVEIS",
    "ARTRODESE CERVICAL ANTERIOR DOIS NIVEIS",
    "ARTRODESE CERVICAL ANTERIOR UM NIVEL",
    "ARTRODESE TORACO-LOMBO-SACRA ANTERIOR DOIS NIVEIS",
    "ARTRODESE TORACO-LOMBO-SACRA POSTERIOR DOIS NIVEIS",
    "TRATAMENTO CIRURGICO DE SINDROME COMPRESSIVA EM TUNEL OSTEO-FIBROSO AO NIVEL DO CARPO",
    "REDUCAO INCRUENTA DE FRATURA / LESAO FISARIA NO PUNHO",
    "REDUCAO INCRUENTA DE LUXACAO OU FRATURA / LUXACAO NO PUNHO",
    "TRATAMENTO CIRURGICO DE DEDO EM GATILHO",
    "TRATAMENTO CIRURGICO DE FRATURA / LESAO FISARIA DAS FALANGES DA MAO (COM FIXACAO)",
    "TRATAMENTO CIRURGICO DE LESAO DA MUSCULATURA INTRINSECA DA MAO",
    "RECONSTRUCAO LIGAMENTAR INTRA-ARTICULAR DO JOELHO (CRUZADO ANTERIOR)",
    "RECONSTRUCAO LIGAMENTAR INTRA-ARTICULAR DO JOELHO (CRUZADO POSTERIOR COM OU SEM ANTERIOR)",
    "TRATAMENTO CIRURGICO DE FRATURA DA PATELA POR FIXACAO INTERNA",
    "AMPUTACAO / DESARTICULACAO DE DEDO",
    "ARTROPLASTIA DE RESSECCAO DE MEDIA / GRANDE ARTICULACAO",
    "MANIPULACAO ARTICULAR",
    "RESSECCAO DE CISTO SINOVIAL",
    "RETIRADA DE FIO OU PINO INTRA-OSSEO",
    "TENOLISE",
    "TENORRAFIA UNICA EM TUNEL OSTEO-FIBROSO",
    "DEBRIDAMENTO DE ULCERA / DE TECIDOS DESVITALIZADOS",
    "HERNIOPLASTIA INCISIONAL",
    "HERNIOPLASTIA INGUINAL (BILATERAL)",
    "HERNIOPLASTIA INGUINAL / CRURAL (UNILATERAL)",
    "HERNIOPLASTIA UMBILICAL"
  ]
},

// ===== UPDATE PADRONIZADO: HOSPITAL CANDIDO RONDON (SSY HOLDING LTDA) =====
// Ajustes aplicados:
// - procedimentos entre aspas (array valido JS)
// - sem acentos (ja esta)
// - "C/ OU S/" e "S/" -> "COM OU SEM" / "SEM"
// - pequenas padronizacoes: "RADIO ULNAR" -> "RADIO-ULNAR", "INTRA-TARSICA" -> "INTRATARSICA"

{
  id: "CANDIDO_RONDON_JIP",
  nome: "HOSPITAL CANDIDO RONDON (SSY HOLDING LTDA)",
  municipio: "JI-PARANÁ",
  tipo: "CIRURGIAS (ORTOPEDIA/UROLOGIA/CLINICA CIRURGICA)",
  perfil: "Hospitalar",
  procedimentos: [
    "AMPUTACAO / DESARTICULACAO DE MEMBROS INFERIORES",
    "AMPUTACAO / DESARTICULACAO DE PE E TARSO",
    "AMPUTACAO COMPLETA ABDOMINO-PERINEAL DO RETO",
    "AMPUTACAO POR PROCIDENCIA DE RETO",
    "ANASTOMOSE BILEO-DIGESTIVA",
    "APENDICECTOMIA",
    "ARTROPLASTIA DE QUADRIL (NAO CONVENCIONAL)",
    "ARTROPLASTIA PARCIAL DE QUADRIL",
    "ARTROPLASTIA TOTAL DE CONVERSAO DE QUADRIL",
    "ARTROPLASTIA TOTAL PRIMARIA DE QUADRIL CIMENTADA",
    "ARTROPLASTIA TOTAL PRIMARIA DE QUADRIL NAO CIMENTADA / HIBRIDA",
    "CAPSULECTOMIA RENAL",
    "CERCLAGEM DE ANUS",
    "CIRURGIA DE UNHA (CANTOPLASTIA)",
    "CISTECTOMIA PARCIAL",
    "CISTOLITOTOMIA E/OU RETIRADA DE CORPO ESTRANHO DA BEXIGA",
    "CISTORRAFIA",
    "CISTOSTOMIA",
    "COLECISTECTOMIA",
    "COLECISTOSTOMIA",
    "COLECTOMIA PARCIAL (HEMICOLECTOMIA)",
    "COLECTOMIA TOTAL",
    "COLEDOCOPLASTIA",
    "COLEDOCOTOMIA COM OU SEM COLECISTECTOMIA",
    "COLOCACAO PERCUTANEA DE CATETER PIELO-URETERO-VESICAL UNILATERAL",
    "COLORRAFIA POR VIA ABDOMINAL",
    "COLOSTOMIA",
    "COLPOPERINEOPLASTIA ANTERIOR E POSTERIOR",
    "CRIPTECTOMIA UNICA / MULTIPLA",
    "CURETAGEM SEMIOTICA",
    "CURETAGEM UTERINA EM MOLA HIDATIFORME",
    "DILATACAO DE COLO DO UTERO",
    "DILATACAO DIGITAL / INSTRUMENTAL DO ANUS E/OU RETO",
    "DILATACAO PERCUTANEA DE ESTENOSES BILIARES",
    "DILATACAO PERCUTANEA DE ESTENOSES URETERAIS",
    "DIVERTICULECTOMIA VESICAL",
    "DRENAGEM BILIAR PERCUTANEA EXTERNA",
    "DRENAGEM BILIAR PERCUTANEA INTERNA",
    "DRENAGEM DE ABSCESSO ANU-RETAL",
    "DRENAGEM DE ABSCESSO ISQUIORRETAL",
    "DRENAGEM DE ABSCESSO PELVICO",
    "DRENAGEM DE ABSCESSO PROSTATICO",
    "DRENAGEM DE ABSCESSO RENAL / PERI-RENAL",
    "DRENAGEM DE ABSCESSO SUBFRENICO",
    "DRENAGEM DE COLECAO PERI-URETRAL",
    "DRENAGEM DE FLEGMAO URINOSO",
    "DRENAGEM DE HEMATOMA / ABSCESSO PRE-PERITONEAL",
    "DRENAGEM DE HEMATOMA / ABSCESSO RETRO-RETAL",
    "ELETROCAUTERIZACAO DE LESAO TRANSPARIETAL DE ANUS",
    "ELETROCOAGULACAO DE LESAO CUTANEA",
    "ENTERECTOMIA",
    "ENTEROPEXIA",
    "ENTEROTOMIA E/OU ENTERORRAFIA",
    "ESFINCTEROTOMIA INTERNA E TRATAMENTO DE FISSURA ANAL",
    "ESPLENECTOMIA",
    "EXCISAO DE LESAO / TUMOR ANU-RETAL",
    "EXCISAO DE LESAO INTESTINAL / MESENTERICA LOCALIZADA",
    "EXCISAO TIPO 3 DO COLO UTERINO",
    "EXCISAO TIPO I DO COLO UTERINO",
    "EXERESE DE CISTO DERMOIDE",
    "EXERESE DE CISTO SACRO-COCCIGEO",
    "EXERESE DE POLIPO DE UTERO",
    "EXTRACAO ENDOSCOPICA DE CALCULO EM PELVE RENAL",
    "EXTRACAO ENDOSCOPICA DE CALCULO EM URETER",
    "FASCIOTOMIA DE MEMBROS INFERIORES",
    "FECHAMENTO DE FISTULA DE COLON",
    "FECHAMENTO DE FISTULA DE RETO",
    "FISTULECTOMIA / FISTULOTOMIA ANAL",
    "HEMORROIDECTOMIA",
    "HEPATECTOMIA PARCIAL",
    "HEPATORRAFIA",
    "HEPATORRAFIA COMPLEXA",
    "HEPATOTOMIA E DRENAGEM DE ABSCESSO",
    "HERNIOPLASTIA DIAFRAGMATICA (VIA ABDOMINAL)",
    "HERNIOPLASTIA DIAFRAGMATICA (VIA TORACICA)",
    "HERNIOPLASTIA EPIGASTRICA",
    "HERNIOPLASTIA INCISIONAL",
    "HERNIOPLASTIA INGUINAL (BILATERAL)",
    "HERNIOPLASTIA INGUINAL / CRURAL (UNILATERAL)",
    "HERNIOPLASTIA RECIDIVANTE",
    "HERNIOPLASTIA UMBILICAL",
    "HERNIORRAFIA COM RESSECCAO INTESTINAL (HERNIA ESTRANGULADA)",
    "HERNIORRAFIA SEM RESSECCAO INTESTINAL (HERNIA ESTRANGULADA)",
    "HISTERECTOMIA COM ANEXECTOMIA",
    "HISTERECTOMIA SUBTOTAL",
    "HISTERECTOMIA TOTAL",
    "HISTERORRAFIA",
    "IMPLANTE DE CATETER URETERAL POR TECNICA CISTOSCOPICA",
    "INJECAO DE GORDURA / TEFLON PERI-URETRAL",
    "INSTALACAO DE TRACAO ESQUELETICA DO MEMBRO INFERIOR",
    "INSTALACAO ENDOSCOPICA DE CATETER DUPLO J",
    "JEJUNOSTOMIA / ILEOSTOMIA",
    "LAPAROTOMIA EXPLORADORA",
    "LAQUEADURA TUBARIA",
    "LAQUEADURA TUBARIA NA MESMA INTERNACAO DE PARTO NORMAL",
    "LIBERACAO DE ADERENCIAS INTESTINAIS",
    "LIGADURA / SECCAO DE VASOS ABERRANTES",
    "LIGADURA ELASTICA DE HEMORROIDAS (SESSAO)",
    "LOMBOTOMIA",
    "MANIPULACAO ARTICULAR",
    "MARSUPIALIZACAO DE ABSCESSO / CISTO",
    "MEATOTOMIA ENDOSCOPICA",
    "MEATOTOMIA SIMPLES",
    "MIOMECTOMIA",
    "NEFRECTOMIA TOTAL",
    "NEFROLITOTOMIA",
    "NEFROLITOTOMIA PERCUTANEA",
    "NEFROPEXIA",
    "NEFROPIELOSTOMIA",
    "NEFRORRAFIA",
    "NEFROSTOMIA (POR PUNCAO)",
    "NEFROSTOMIA COM OU SEM DRENAGEM",
    "NEFROSTOMIA PERCUTANEA",
    "NEFROURETERECTOMIA TOTAL",
    "OOFORECTOMIA / OOFOROPLASTIA",
    "OSTECTOMIA DE OSSOS DA MAO E/OU DO PE",
    "OSTECTOMIA DE OSSOS LONGOS EXCETO DA MAO E DO PE",
    "OSTEOTOMIA DE OSSOS DA MAO E/OU DO PE",
    "OSTEOTOMIA DE OSSOS LONGOS EXCETO DA MAO E DO PE",
    "PANCREATECTOMIA PARCIAL",
    "PANCREATOTOMIA PARA DRENAGEM",
    "PARACENTESE ABDOMINAL",
    "PATELECTOMIA TOTAL OU PARCIAL",
    "PERITONIOSTOMIA COM TELA INORGANICA",
    "PIELOLITOTOMIA",
    "PIELOPLASTIA",
    "PIELOTOMIA",
    "PLASTICA ANAL EXTERNA / ESFINCTEROPLASTIA ANAL",
    "PNEUMOPERITONIO (POR SESSAO)",
    "PROCTOCOLECTOMIA TOTAL COM RESERVATORIO ILEAL",
    "PROCTOPEXIA ABDOMINAL POR PROCIDENCIA DO RETO",
    "PROCTOPLASTIA E PROCTORRAFIA POR VIA PERINEAL",
    "PROSTATECTOMIA SUPRAPUBICA",
    "PUNCAO / ASPIRACAO DA BEXIGA",
    "REDUCAO CIRURGICA DE VOLVO POR LAPAROTOMIA",
    "REDUCAO INCRUENTA DE FRATURA / LUXACAO / FRATURA-LUXACAO DO TORNOZELO",
    "REDUCAO INCRUENTA DE FRATURA DIAFISARIA / LESAO FISARIA DISTAL DA TIBIA COM OU SEM FRATURA DA FIBULA",
    "REDUCAO INCRUENTA DE FRATURA DIAFISARIA / LESAO FISARIA PROXIMAL DO FEMUR",
    "REDUCAO INCRUENTA DE LUXACAO OU FRATURA / LUXACAO SUBTALAR E INTRATARSICA",
    "REDUCAO INCRUENTA DE LUXACAO OU FRATURA / LUXACAO TARSO-METATARSICA",
    "REDUCAO MANUAL DE PROCIDENCIA DE RETO",
    "REMOCAO CIRURGICA DE FECALOMA",
    "REPARACAO DE OUTRAS HERNIAS",
    "RESSECCAO DE CARUNCULA URETRAL",
    "RESSECCAO DE CISTO SINOVIAL",
    "RESSECCAO DE PROLAPSO DA MUCOSA DA URETRA",
    "RESSECCAO DE VARIZES PELVICAS",
    "RESSECCAO DO COLO VESICAL / TUMOR VESICAL A CEU ABERTO",
    "RESSECCAO DO EPIPLOM",
    "RESSECCAO E FECHAMENTO DE FISTULA URETRAL",
    "RESSECCAO ENDOSCOPICA DA EXTREMIDADE DISTAL DO URETER",
    "RESSECCAO ENDOSCOPICA DE LESAO VESICAL",
    "RESSECCAO ENDOSCOPICA DE PROSTATA",
    "RESSUTURA DE PAREDE ABDOMINAL (POR DEISCENCIA TOTAL / EVISCERACAO)",
    "RETIRADA DE CORPO ESTRANHO / POLIPOS DO RETO / COLO SIGMOIDE",
    "RETIRADA DE CORPO ESTRANHO INTRA-ARTICULAR",
    "RETIRADA DE CORPO ESTRANHO INTRA-OSSEO",
    "RETIRADA DE ESPACADORES / OUTROS MATERIAIS",
    "RETIRADA DE FIO OU PINO INTRA-OSSEO",
    "RETIRADA DE FIXADOR EXTERNO",
    "RETIRADA DE PLACA E/OU PARAFUSOS",
    "RETIRADA DE TRACAO TRANS-ESQUELETICA",
    "RETIRADA PERCUTANEA DE CALCULO URETERAL COM CATETER",
    "RETIRADA PERCUTANEA DE CALCULOS BILIARES",
    "RETOSSIGMOIDECTOMIA ABDOMINAL",
    "REVISAO CIRURGICA DE COTO DE AMPUTACAO EM MEMBRO INFERIOR (EXCETO DEDOS DO PE)",
    "SALPINGECTOMIA UNI / BILATERAL",
    "SALPINGOPLASTIA",
    "SINDACTILIA CIRURGICA DOS DEDOS DO PE (PROCEDIMENTO TIPO KELIKIAN)",
    "TALECTOMIA",
    "TENODESE",
    "TENOLISE",
    "TENOMIORRAFIA",
    "TRAQUELOPLASTIA",
    "TRATAMENTO CIRURGICO DE ARTRITE INFECCIOSA (GRANDES E MEDIAS ARTICULACOES)",
    "TRATAMENTO CIRURGICO DE ARTRITE INFECCIOSA DAS PEQUENAS ARTICULACOES",
    "TRATAMENTO CIRURGICO DE AVULSAO DO GRANDE E DO PEQUENO TROCANTER",
    "TRATAMENTO CIRURGICO DE CISTO DE RIM POR PUNCAO",
    "TRATAMENTO CIRURGICO DE CISTOCELE",
    "TRATAMENTO CIRURGICO DE CISTOS PANCREATICOS",
    "TRATAMENTO CIRURGICO DE COALIZAO TARSAL",
    "TRATAMENTO CIRURGICO DE FISTULA URETRO-VAGINAL",
    "TRATAMENTO CIRURGICO DE FISTULA VESICO-VAGINAL",
    "TRATAMENTO CIRURGICO DE FRATURA / LESAO FISARIA PROXIMAL (COLO) DO FEMUR (SINTESE)",
    "TRATAMENTO CIRURGICO DE FRATURA SUPRACONDILEANA DO FEMUR (METAFISE DISTAL)",
    "TRATAMENTO CIRURGICO DE HALUX VALGUS COM OSTEOTOMIA DO PRIMEIRO OSSO METATARSIANO",
    "TRATAMENTO CIRURGICO DE HEMORRAGIA VESICAL (FORMOLIZACAO DA BEXIGA)",
    "TRATAMENTO CIRURGICO DE INCONTINENCIA URINARIA",
    "TRATAMENTO CIRURGICO DE INCONTINENCIA URINARIA POR VIA VAGINAL",
    "TRATAMENTO CIRURGICO DE INCONTINENCIA URINARIA VIA ABDOMINAL",
    "TRATAMENTO CIRURGICO DE LESAO AGUDA CAPSULO-LIGAMENTAR MEMBRO INFERIOR (JOELHO / TORNOZELO)",
    "TRATAMENTO CIRURGICO DE PERITONITE",
    "TRATAMENTO CIRURGICO DE REFLUXO VESICO-URETERAL",
    "TRATAMENTO CIRURGICO DE ROTURA / DESINSERCAO / ARRANCAMENTO CAPSULO-TENO-LIGAMENTAR NA MAO",
    "TRATAMENTO CIRURGICO DE ROTURA DO MENISCO COM MENISCECTOMIA PARCIAL / TOTAL",
    "TRATAMENTO CIRURGICO DE SINDACTILIA DA MAO (POR ESPACO INTERDIGITAL)",
    "TRATAMENTO CIRURGICO DE SINOSTOSE RADIO-ULNAR",
    "TRATAMENTO CIRURGICO DE URETEROCELE",
    "TRATAMENTO CIRURGICO DO HALUX RIGIDUS",
    "TRATAMENTO CIRURGICO DO HALUX VALGUS SEM OSTEOTOMIA DO PRIMEIRO OSSO METATARSIANO",
    "TRATAMENTO CIRURGICO PARA CENTRALIZACAO DO PUNHO",
    "TRATAMENTO DAS LESOES OSTEO-CONDRAIS POR FIXACAO OU MOSAICOPLASTIA JOELHO / TORNOZELO",
    "URETERECTOMIA",
    "URETEROENTEROSTOMIA",
    "URETEROLITOTOMIA",
    "URETEROLITOTRIPSIA TRANSURETEROSCOPICA",
    "URETEROPLASTIA",
    "URETEROSTOMIA CUTANEA",
    "URETROPLASTIA (RESSECCAO DE CORDA)",
    "URETROPLASTIA AUTOGENA",
    "URETROPLASTIA HETEROGENEA",
    "URETRORRAFIA",
    "URETROSTOMIA PERINEAL / CUTANEA / EXTERNA",
    "URETROTOMIA INTERNA",
    "URETROTOMIA PARA RETIRADA DE CALCULO OU CORPO ESTRANHO"
  ]
},
{
  id: "CIR_CARDI_ADULTO_JI_PARANA_CANDIDO_RONDON",
  nome: "HOSPITAL CANDIDO RONDON (SSY HOLDING LTDA)",
  municipio: "JI-PARANÁ",
  tipo: "CIRURGIA CARDIACA - ADULTO",
  perfil: "Hospitalar",
  link: "https://drive.google.com/file/d/1ev6UCrRkEs8vs-mC9YeLMUG51yN9s8YK/view?usp=sharing",
  procedimentos: [
    "ABERTURA DE ESTENOSE AORTICA VALVAR",
    "FECHAMENTO DE COMUNICACAO INTERATRIAL",
    "FECHAMENTO DE COMUNICACAO INTERVENTRICULAR",
    "IMPLANTE DE PROTESE VALVAR",
    "PERICARDIECTOMIA",
    "PLASTICA VALVAR",
    "PLASTICA VALVAR COM REVASCULARIZACAO MIOCARDICA",
    "PLASTICA VALVAR E/OU TROCA VALVAR MULTIPLA",
    "RECONSTRUCAO DA RAIZ DA AORTA",
    "RECONSTRUCAO DA RAIZ DA AORTA COM TUBO VALVADO",
    "REVASCULARIZACAO MIOCARDICA COM USO DE EXTRACORPOREA",
    "REVASCULARIZACAO MIOCARDICA SEM USO DE EXTRACORPOREA",
    "REVASCULARIZACAO MIOCARDICA SEM USO DE EXTRACORPOREA (COM 2 OU MAIS ENXERTOS)",
    "TROCA VALVAR COM REVASCULARIZACAO MIOCARDICA",
    "CORRECAO DE PERSISTENCIA DO CANAL ARTERIAL",
    "CORRECAO DE INSUFICIENCIA DA VALVULA TRICUSPIDE",
    "CORRECAO DE COMUNICACAO INTERVENTRICULAR",
    "CORRECAO DE ANEURISMA / DISSECCAO DA AORTA TORACO-ABDOMINAL",
    "RESSECCAO DE TUMOR INTRACARDIACO",
    "REVASCULARIZACAO MIOCARDICA COM USO DE EXTRACORPOREA (COM 2 OU MAIS ENXERTOS)",
    "TROCA DE AORTA ASCENDENTE",
    "IMPLANTE COM TROCA DE POSICAO DE VALVAS (CIRURGIA DE ROSS)",
    "MEDIASTINOTOMIA EXPLORADORA PARA-ESTERNAL / POR VIA ANTERIOR",
    "TORACOSTOMIA COM DRENAGEM PLEURAL FECHADA",

    "IMPLANTE DE MARCAPASSO DE CAMARA UNICA TRANSVENOSO",
    "IMPLANTE DE CARDIOVERSOR DESFIBRILADOR DE CAMARA UNICA TRANSVENOSO",
    "IMPLANTE DE CARDIOVERSOR DESFIBRILADOR DE CAMARA DUPLA TRANSVENOSO",
    "IMPLANTE DE MARCAPASSO CARDIACO MULTI-SITIO TRANSVENOSO",
    "IMPLANTE DE MARCAPASSO DE CAMARA DUPLA TRANSVENOSO",
    "IMPLANTE DE CARDIOVERSOR DESFIBRILADOR (CDI) MULTI-SITIO TRANSVENOSO",

    "TROCA DE GERADOR DE MARCAPASSO DE CAMARA UNICA",
    "TROCA DE GERADOR DE MARCAPASSO DE CAMARA DUPLA",
    "TROCA DE GERADOR DE CARDIO-DESFIBRILADOR DE CAMARA UNICA / DUPLA",
    "TROCA DE GERADOR DE MARCAPASSO MULTI-SITIO",

    "ESTUDO ELETROFISIOLOGICO DIAGNOSTICO",
    "ESTUDO ELETROFISIOLOGICO TERAPEUTICO I (ABLACAO DE FLUTTER ATRIAL)",
    "ESTUDO ELETROFISIOLOGICO TERAPEUTICO I",
    "ESTUDO ELETROFISIOLOGICO TERAPEUTICO II (ABLACAO DAS VIAS ANOMALAS MULTIPLAS)",
    "ESTUDO ELETROFISIOLOGICO TERAPEUTICO II (ABLACAO DE FIBRILACAO ATRIAL)",
    "ESTUDO ELETROFISIOLOGICO TERAPEUTICO II (ABLACAO DE TAQUICARDIA VENTRICULAR SUSTENTADA COM CARDIOPATIA ESTRUTURAL)",

    "EXTRACAO PERCUTANEA DE ELETRODOS"
  ]
},

  {
  id: "CIR_CARDI_PED_JI_PARANA_CANDIDO_RONDON",
  nome: "HOSPITAL CANDIDO RONDON (SSY HOLDING LTDA)",
  municipio: "JI-PARANÁ",
  tipo: "CIRURGIA CARDIACA - PEDIATRICA",
  perfil: "Hospitalar",
  link: "",
  procedimentos: [
    "ABERTURA DE COMUNICACAO INTERATRIAL",
    "ABERTURA DE ESTENOSE AORTICA VALVAR",
    "FECHAMENTO DE COMUNICACAO INTERATRIAL",
    "ANASTOMOSE CAVO-PULMONAR BIDIRECIONAL",
    "ANASTOMOSE CAVO-PULMONAR TOTAL",
    "BANDAGEM DA ARTERIA PULMONAR",
    "CORRECAO DE ATRESIA PULMONAR",
    "CORRECAO DE COR TRIATRIATUM",
    "CORRECAO DE DRENAGEM ANOMALA TOTAL DE VEIAS PULMONARES",
    "CORRECAO DE DUPLA VIA DE SAIDA DO VENTRICULO DIREITO",
    "CORRECAO DE DUPLA VIA DE SAIDA DO VENTRICULO ESQUERDO",
    "CORRECAO DE HIPOPLASIA DE VENTRICULO ESQUERDO",
    "CORRECAO DE INSUFICIENCIA DA VALVULA TRICUSPIDE",
    "CORRECAO DE INSUFICIENCIA MITRAL CONGENITA",
    "CORRECAO DE PERSISTENCIA DO CANAL ARTERIAL",
    "CORRECAO DE TETRALOGIA DE FALLOT E VARIANTES",
    "CORRECAO DE TRANSPOSICAO DOS GRANDES VASOS DA BASE (CRIANCA E ADOLESCENTE)",
    "CORRECAO DE TRONCO ARTERIOSO PERSISTENTE",
    "CORRECAO DO CANAL ATRIO-VENTRICULAR (TOTAL)",
    "FECHAMENTO DE COMUNICACAO INTERVENTRICULAR",
    "IMPLANTE COM TROCA DE POSICAO DE VALVAS (CIRURGIA DE ROSS)",
    "IMPLANTE DE PROTESE VALVAR",
    "PERICARDIECTOMIA",
    "PLASTICA VALVAR",
    "PLASTICA VALVAR COM REVASCULARIZACAO MIOCARDICA",
    "PLASTICA VALVAR E/OU TROCA VALVAR MULTIPLA",
    "PLASTICA / TROCA DE VALVULA TRICUSPIDE (ANOMALIA DE EBSTEIN)",

    "ESTUDO ELETROFISIOLOGICO DIAGNOSTICO",
    "ESTUDO ELETROFISIOLOGICO TERAPEUTICO I (ABLACAO DE FLUTTER ATRIAL)",
    "ESTUDO ELETROFISIOLOGICO TERAPEUTICO I",
    "ESTUDO ELETROFISIOLOGICO TERAPEUTICO II (ABLACAO DAS VIAS ANOMALAS MULTIPLAS)",
    "ESTUDO ELETROFISIOLOGICO TERAPEUTICO II (ABLACAO DE FIBRILACAO ATRIAL)",
    "ESTUDO ELETROFISIOLOGICO TERAPEUTICO II (ABLACAO DE TAQUICARDIA VENTRICULAR SUSTENTADA COM CARDIOPATIA ESTRUTURAL)",

    "TROCA VALVAR COM REVASCULARIZACAO MIOCARDICA",
    "RESSECCAO DE TUMOR INTRACARDIACO",
    "MEDIASTINOTOMIA EXPLORADORA PARA-ESTERNAL / POR VIA ANTERIOR",
    "TORACOSTOMIA COM DRENAGEM PLEURAL FECHADA",

    "IMPLANTE DE CARDIOVERSOR DESFIBRILADOR DE CAMARA UNICA TRANSVENOSO",
    "IMPLANTE DE CARDIOVERSOR DESFIBRILADOR DE CAMARA DUPLA TRANSVENOSO",
    "IMPLANTE DE CARDIOVERSOR DESFIBRILADOR (CDI) MULTI-SITIO TRANSVENOSO",
    "TROCA DE GERADOR DE MARCAPASSO DE CAMARA DUPLA",
    "TROCA DE GERADOR DE CARDIO-DESFIBRILADOR DE CAMARA UNICA / DUPLA",
    "TROCA DE GERADOR DE MARCAPASSO MULTI-SITIO"
  ]
},

 {
  id: "CIR_CARDI_NEO_JI_PARANA_CANDIDO_RONDON",
  nome: "HOSPITAL CANDIDO RONDON (SSY HOLDING LTDA)",
  municipio: "JI-PARANÁ",
  tipo: "CIRURGIA CARDIACA - NEONATAL",
  perfil: "Hospitalar",
  link: "",
  procedimentos: [
    "ABERTURA DE COMUNICACAO INTERATRIAL",
    "ABERTURA DE ESTENOSE AORTICA VALVAR",
    "ABERTURA DE ESTENOSE PULMONAR VALVAR",
    "FECHAMENTO DE COMUNICACAO INTERATRIAL",
    "ANASTOMOSE CAVO-PULMONAR BIDIRECIONAL",
    "ANASTOMOSE CAVO-PULMONAR TOTAL",
    "BANDAGEM DA ARTERIA PULMONAR",
    "CORRECAO DE ATRESIA PULMONAR",
    "CORRECAO DE COR TRIATRIATUM",
    "CORRECAO DE DRENAGEM ANOMALA TOTAL DE VEIAS PULMONARES",
    "CORRECAO DE DUPLA VIA DE SAIDA DO VENTRICULO DIREITO",
    "CORRECAO DE DUPLA VIA DE SAIDA DO VENTRICULO ESQUERDO",
    "CORRECAO DE HIPOPLASIA DE VENTRICULO ESQUERDO",
    "CORRECAO DE INSUFICIENCIA DA VALVULA TRICUSPIDE",
    "CORRECAO DE INSUFICIENCIA MITRAL CONGENITA",
    "CORRECAO DE PERSISTENCIA DO CANAL ARTERIAL",
    "CORRECAO DE TETRALOGIA DE FALLOT E VARIANTES",
    "CORRECAO DE TRANSPOSICAO DOS GRANDES VASOS DA BASE (CRIANCAS E ADOLESCENTES)",
    "CORRECAO DE TRONCO ARTERIOSO PERSISTENTE",
    "CORRECAO DO CANAL ATRIO-VENTRICULAR (TOTAL)",
    "FECHAMENTO DE COMUNICACAO INTERVENTRICULAR",
    "IMPLANTE COM TROCA DE POSICAO DE VALVAS (CIRURGIA DE ROSS)",
    "IMPLANTE DE MARCAPASSO DE CAMARA UNICA TRANSVENOSO"
  ]
},

  /* --- DEMAIS UNIDADES (cadastro pronto; rol a preencher depois) --- */
  {
    id: "CIR_PORTO_VELHO_SAMAR",
    nome: "HOSPITAL SAMAR - PORTO VELHO",
    municipio: "PORTO VELHO",
    tipo: "CIRURGIAS",
    perfil: "Hospitalar",
    link: "https://drive.google.com/file/d/1QKUEnbgspA2ht1ByXny9tiqCJu1ugz6Y/view?usp=sharing",
    procedimentos: [
      "COLECISTECTOMIA",
      "HERNIOPLASTIA EPIGASTRICA",
      "HERNIOPLASTIA INCISIONAL",
      "HERNIOPLASTIA INGUINAL (BILATERAL)",
      "HERNIOPLASTIA INGUINAL / CRURAL (UNILATERAL)",
      "HERNIOPLASTIA RECIDIVANTE",
      "HERNIOPLASTIA UMBILICAL",
      "HERNIORRAFIA S/ RESSECCAO INTESTINAL (HERNIA ESTRANGULADA)"
    ]
  },

  {
    id: "CIR_PORTO_VELHO_9_DE_JULHO",
    nome: "HOSPITAL 9 DE JULHO DE RONDONIA LTDA",
    municipio: "PORTO VELHO",
    tipo: "CIRURGIAS",
    perfil: "Hospitalar",
    link: "",
    procedimentos: [
      "ARTROPLASTIA TOTAL DE CONVERSAO DE QUADRIL",
      "ARTROPLASTIA PARCIAL DE QUADRIL",
      "ARTROPLASTIA TOTAL PRIMARIA DE QUADRIL CIMENTADA",
      "ARTROPLASTIA TOTAL PRIMARIA DE QUADRIL NAO CIMENTADA / HIBRIDA",
      "RETIRADA DE FIXADOR EXTERNO",
      "RETIRADA DE FIO OU PINO INTRA-OSSEO",
      "REDUCAO INCRUENTA DE FRATURA DE DIAFISE DO UMERO",
      "REDUCAO INCRUENTA DE FRATURA DIAFISARIA DOS OSSOS DO ANTEBRACO",
      "REDUCAO INCRUENTA DE LUXACAO / FRATURA-LUXACAO DO COTOVELO",
      "REDUCAO INCRUENTA DE LUXACAO OU FRATURA / LUXACAO NO PUNHO",
      "CIRURGIA DE UNHA (CANTOPLASTIA)",
      "EXCISAO E ENXERTO DE PELE (HEMANGIOMA, NEVUS OU TUMOR)",
      "EXTIRPACAO E SUPRESSAO DE LESAO DE PELE E DE TECIDO CELULAR SUBCUTANEO",
      "BIOPSIA / PUNCAO DE TUMOR SUPERFICIAL DA PELE",
      "BIOPSIA DE BOLSA ESCROTAL",
      "BIOPSIA DE CONDUTO AUDITIVO EXTERNO",
      "BIOPSIA DE ENDOMETRIO",
      "BIOPSIA DE MUSCULO (A CEU ABERTO)",
      "BIOPSIA DE FIGADO POR PUNCAO",
      "BIOPSIA DE GANGLIO LINFATICO",
      "BIOPSIA DE LESAO DE PARTES MOLES (POR AGULHA / CEU ABERTO)",
      "BIOPSIA DE PELE E PARTES MOLES",
      "BIOPSIA DE TIREOIDE OU PARATIREOIDE - PAAF",
      "PUNCAO ASPIRATIVA DE MAMA POR AGULHA FINA",
      "PUNCAO DE MAMA POR AGULHA GROSSA",
      "BIOPSIA DO COLO UTERINO",
      "IMPLANTACAO DE CATETER DE LONGA PERMANENCIA SEMI OU TOTALMENTE IMPLANTAVEL",
      "RETIRADA DE CATETER DE LONGA PERMANENCIA SEMI OU TOTALMENTE IMPLANTAVEL",
      "DRENAGEM DE GANGLIO LINFATICO",
      "EXCISAO E SUTURA DE LINFANGIOMA / NEVUS",
      "INSTALACAO DE CATETER VENOSO DE LONGA PERMANENCIA TOTALMENTE IMPLANTAVEL",
      "CONFECCAO DE FISTULA ARTERIOVENOSA P/ ACESSO",
      "EXCISAO E SUTURA DE HEMANGIOMA",
      "EXERESE DE GANGLIO LINFATICO",
      "LINFADENECTOMIA RADICAL INGUINAL BILATERAL",
      "LINFADENECTOMIA RADICAL INGUINAL UNILATERAL",
      "LINFADENECTOMIA RADICAL AXILAR UNILATERAL",
      "TRATAMENTO CIRURGICO DE VARIZES (UNILATERAL)",
      "TRATAMENTO CIRURGICO DE VARIZES (BILATERAL)",
      "PARACENTESE ABDOMINAL"
    ]
  },

  { id: "CIR_PORTO_VELHO_GATE", nome: "GATE-SERVICOS MEDICO HOSPITALARES LTDA", municipio: "PORTO VELHO", tipo: "CIRURGIAS", link: "", procedimentos: [] },
  { id: "CIR_PORTO_VELHO_SANTA_MARCELINA", nome: "CASA DE SAUDE SANTA MARCELINA", municipio: "PORTO VELHO", tipo: "CIRURGIAS", link: "", procedimentos: [] },
  { id: "CIR_PORTO_VELHO_PRONTOCORDIS", nome: "HOSP-COR HOSPITAL DO CORACAO DE RONDONIA (PRONTOCORDIS)", municipio: "PORTO VELHO", tipo: "CIRURGIAS", link: "https://drive.google.com/file/d/1jvCO6TJ7GyybErP24GrIpPtKonO2hL5m/view?usp=sharing", procedimentos: [] },
  { id: "CIR_PORTO_VELHO_IRB_PRIME", nome: "IRB PRIME CARE SERVICOS MEDICOS CLINICO HOSPITALAR LTDA", municipio: "PORTO VELHO", tipo: "CIRURGIAS", link: "", procedimentos: [] },

  {
    id: "CIR_PORTO_VELHO_INSTITUTO_CORACAO",
    nome: "INSTITUTO DO CORACAO DE RONDONIA",
    municipio: "PORTO VELHO",
    tipo: "CIRURGIAS",
    perfil: "Hospitalar",
    link: "https://drive.google.com/file/d/1jvCO6TJ7GyybErP24GrIpPtKonO2hL5m/view?usp=sharing",
    procedimentos: [
      "BIOPSIA DE OSSO / CARTILAGEM DE MEMBRO SUPERIOR (POR AGULHA / CEU ABERTO)",
      "TRATAMENTO CIRURGICO DE NEUROPATIA COMPRESSIVA COM OU SEM MICROCIRURGIA",
      "TRATAMENTO CIRURGICO DE SINDROME COMPRESSIVA EM TUNEL OSTEO-FIBROSO AO NIVEL DO CARPO",
      "TRATAMENTO CIRURGICO DE VARIZES (BILATERAL)",
      "REDUCAO INCRUENTA DE LUXACAO OU FRATURA / LUXACAO ESCAPULO-UMERAL",
      "TRATAMENTO CIRURGICO DE VARIZES (UNILATERAL)",
      "TRATAMENTO CIRURGICO DE FRATURA DA CLAVICULA",
      "TRATAMENTO CIRURGICO DE LUXACAO / FRATURA-LUXACAO ACROMIO-CLAVICULAR",
      "REDUCAO INCRUENTA DE FRATURA / LESAO FISARIA NO PUNHO",
      "REDUCAO INCRUENTA DE FRATURA DA DIAFISE DO UMERO",
      "TENOSINOVECTOMIA EM MEMBRO SUPERIOR",
      "TRATAMENTO CIRURGICO DE DEDO EM GATILHO",
      "RESSECCAO DE CISTO SINOVIAL",
      "RESSECCAO SIMPLES DE TUMOR OSSEO / DE PARTES MOLES",
      "TRATAMENTO CIRURGICO DE HERNIA MUSCULAR",
      "ARTROPLASTIA PARCIAL DE QUADRIL",
      "ARTROPLASTIA TOTAL DE CONVERSAO DE QUADRIL",
      "ARTROPLASTIA DE REVISAO OU RECONSTRUCAO DE QUADRIL",
      "ARTROPLASTIA TOTAL PRIMARIA DE QUADRIL CIMENTADA",
      "ARTROPLASTIA TOTAL PRIMARIA DE QUADRIL NAO CIMENTADA / HIBRIDA",
      "AMPUTACAO / DESARTICULACAO DE MEMBROS INFERIORES",
      "AMPUTACAO / DESARTICULACAO DE PE E TARSO",
      "FASCIOTOMIA DE MEMBROS INFERIORES",
      "REDUCAO INCRUENTA DA LUXACAO / FRATURA-LUXACAO METATARSO-FALANGIANA / INTERFALANGIANA DO PE",
      "TRATAMENTO CIRURGICO DE FRATURA / LESAO FISARIA DOS METATARSIANOS",
      "TRATAMENTO CIRURGICO DE FRATURA / LESAO FISARIA DOS PODODACTILOS",
      "AMPUTACAO / DESARTICULACAO DE DEDO",
      "RETIRADA DE CORPO ESTRANHO INTRA-OSSEO",
      "RETIRADA DE FIO OU PINO INTRA-OSSEO",
      "RETIRADA DE FIXADOR EXTERNO",
      "TENOLISE",
      "DEBRIDAMENTO DE FASCEITE NECROTIZANTE",
      "DEBRIDAMENTO DE ULCERA / DE TECIDOS DESVITALIZADOS"
    ]
  },

  {
    id: "CIR_HB",
    nome: "HOSPITAL DE BASE DR. ARY PINHEIRO",
    municipio: "PORTO VELHO",
    tipo: "CIRURGIAS",
    link: "",
    procedimentos: ["ADENOMASTECTOMIA"]
  },

  /* ===== REDE HOSPITALAR ===== */
  {
    id: "DIAG_HB",
    nome: "HOSPITAL DE BASE DR. ARY PINHEIRO",
    municipio: "PORTO VELHO",
    tipo: "CIRURGIAS",
    perfil: "Ambulatorial e Hospitalar",
    link: "",
    procedimentos: [
      "ANGIOGRAFIA DE ARCO AORTICO",
      "ANGIOPLASTIA INTRALUMINAL DE VASOS DAS EXTREMIDADES (SEM STENT)",
      "ANGIOPLASTIA INTRALUMINAL DOS VASOS DO PESCOCO / TRONCOS SUPRA-AORTICOS (COM STENT RECOBERTO)",
      "APLICACAO DE TESTE P/ PSICODIAGNOSTICO",
      "ARTERIOGRAFIA SELETIVA POR CATETER (POR VASO)",
      "BIOPSIA DE FIGADO POR PUNCAO",
      "BIOPSIA DE PROSTATA",
      "BIOPSIA DE PULMAO POR ASPIRACAO",
      "BIOPSIA DE RIM POR PUNCAO",
      "BIOPSIA DE TIREOIDE OU PARATIREOIDE - PAAF",
      "BRONCOSCOPIA (BRONCOFIBROSCOPIA)",
      "CATETERISMO CARDIACO",
      "CLISTER OPACO C/ DUPLO CONTRASTE",
      "COLONOSCOPIA (COLOSCOPIA)",
      "CONSULTA CIRURGIA VASCULAR - TRATAMENTO DE VARIZES COM ESPUMA NAO ESTETICA",
      "CONSULTA EM CARDIOLOGIA",
      "CONSULTA EM CARDIOLOGIA - CIRURGIA CARDIACA",
      "CONSULTA EM CARDIOLOGIA - MARCAPASSO",
      "CONSULTA EM CIRURGIA BARIATRICA",
      "CONSULTA EM CIRURGIA DA CABECA E PESCOCO",
      "CONSULTA EM CIRURGIA GERAL",
      "CONSULTA EM CIRURGIA PLASTICA",
      "CONSULTA EM CIRURGIA TORACICA",
      "CONSULTA EM CLINICA MEDICA",
      "CONSULTA EM ENFERMAGEM - BARIATRICA",
      "CONSULTA EM HEMATOLOGIA - GERAL",
      "CONSULTA EM HEPATOLOGIA",
      "CONSULTA EM MASTOLOGIA",
      "CONSULTA EM NUTRICAO - BARIATRICA",
      "CONSULTA EM PNEUMOLOGIA - GERAL",
      "CONSULTA EM TRIAGEM HEMODINAMICA",
      "CONSULTA EM UROLOGIA - ADULTO",
      "CONSULTA ESPECIALIZADA EM PEDIATRIA 1A CONSULTA",
      "ECOCARDIOGRAFIA TRANSESOFAGICA",
      "ECOCARDIOGRAFIA TRANSTORACICA",
      "ELETROCARDIOGRAMA",
      "ESCANOMETRIA",
      "ESOFAGOGASTRODUODENOSCOPIA",
      "ESPIROMETRIA OU PROVA DE FUNCAO PULMONAR COMPLETA COM BRONCODILATADOR",
      "HEMODIALISE P/ PACIENTES RENAIS AGUDOS / CRONICOS AGUDIZADOS S/ TRATAMENTO DIALITICO INICIADO",
      "MARCACAO DE LESAO PRE-CIRURGICA DE LESAO NAO PALPAVEL DE MAMA ASSOCIADA A ULTRASSONOGRAFIA",
      "BIOPSIA DE TIREOIDE OU PARATIREOIDE - PAAF",
      "PULSOTERAPIA II (POR APLICACAO)",
      "RADIOGRAFIA BILATERAL DE ORBITAS (PA + OBLIQUAS + HIRTZ)",
      "RADIOGRAFIA DE ABDOMEN (AP + LATERAL / LOCALIZADA)",
      "RADIOGRAFIA DE ABDOMEN AGUDO (MINIMO DE 3 INCIDENCIAS)",
      "RADIOGRAFIA DE ABDOMEN SIMPLES (AP)",
      "RADIOGRAFIA DE ANTEBRACO",
      "RADIOGRAFIA DE ARCADA ZIGOMATICO-MALAR (AP + OBLIQUAS)",
      "RADIOGRAFIA DE ARTICULACAO ACROMIO-CLAVICULAR",
      "RADIOGRAFIA DE ARTICULACAO COXO-FEMORAL",
      "RADIOGRAFIA DE ARTICULACAO ESCAPULO-UMERAL",
      "RADIOGRAFIA DE ARTICULACAO SACRO-ILIACA",
      "RADIOGRAFIA DE ARTICULACAO TEMPORO-MANDIBULAR BILATERAL",
      "RADIOGRAFIA DE ARTICULACAO TIBIO-TARSICA",
      "RADIOGRAFIA DE BACIA",
      "RADIOGRAFIA DE BRACO",
      "RADIOGRAFIA DE CALCANEO",
      "RADIOGRAFIA DE CAVUM (LATERAL + HIRTZ)",
      "RADIOGRAFIA DE CLAVICULA",
      "RADIOGRAFIA DE COLUNA CERVICAL (AP + LATERAL + TO / FLEXAO)",
      "RADIOGRAFIA DE COLUNA CERVICAL (AP + LATERAL + TO + OBLIQUAS)",
      "RADIOGRAFIA DE COLUNA CERVICAL FUNCIONAL / DINAMICA",
      "RADIOGRAFIA DE COLUNA LOMBO-SACRA",
      "RADIOGRAFIA DE COLUNA LOMBO-SACRA (C/ OBLIQUAS)",
      "RADIOGRAFIA DE COLUNA LOMBO-SACRA FUNCIONAL / DINAMICA",
      "RADIOGRAFIA DE COLUNA TORACICA (AP + LATERAL)",
      "RADIOGRAFIA DE COLUNA TORACO-LOMBAR",
      "RADIOGRAFIA DE COLUNA TORACO-LOMBAR DINAMICA",
      "RADIOGRAFIA DE COSTELAS (POR HEMITORAX)",
      "RADIOGRAFIA DE COTOVELO",
      "RADIOGRAFIA DE COXA",
      "RADIOGRAFIA DE CRANIO (PA + LATERAL + OBLIQUA / BRETTON + HIRTZ)",
      "RADIOGRAFIA DE CRANIO (PA + LATERAL)",
      "RADIOGRAFIA DE ESCAPULA/OMBRO (TRES POSICOES)",
      "RADIOGRAFIA DE ESOFAGO",
      "RADIOGRAFIA DE ESTOMAGO E DUODENO",
      "RADIOGRAFIA DE INTESTINO DELGADO (TRANSITO)",
      "RADIOGRAFIA DE JOELHO (AP + LATERAL)",
      "RADIOGRAFIA DE JOELHO OU PATELA (AP + LATERAL + AXIAL)",
      "RADIOGRAFIA DE JOELHO OU PATELA (AP + LATERAL + OBLIQUA + 3 AXIAIS)",
      "RADIOGRAFIA DE MAO",
      "RADIOGRAFIA DE MAO E PUNHO (P/ DETERMINACAO DE IDADE OSSEA)",
      "RADIOGRAFIA DE MAXILAR (PA + OBLIQUA)",
      "RADIOGRAFIA DE OSSOS DA FACE (MN + LATERAL + HIRTZ)",
      "RADIOGRAFIA DE PE / DEDOS DO PE",
      "RADIOGRAFIA DE PERNA",
      "RADIOGRAFIA DE PUNHO (AP + LATERAL + OBLIQUA)",
      "RADIOGRAFIA DE REGIAO SACRO-COCCIGEA",
      "RADIOGRAFIA DE SEIOS DA FACE (FN + MN + LATERAL + HIRTZ)",
      "RADIOGRAFIA DE TORAX (PA + INSPIRACAO + EXPIRACAO + LATERAL)",
      "RADIOGRAFIA DE TORAX (PA + LATERAL + OBLIQUA)",
      "RADIOGRAFIA DE TORAX (PA E PERFIL)",
      "RADIOGRAFIA DE TORAX (PA)",
      "RADIOGRAFIA PANORAMICA DE MEMBROS INFERIORES",
      "TOMOGRAFIA COMPUTADORIZADA DE ABDOMEN SUPERIOR",
      "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO INFERIOR",
      "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO SUPERIOR",
      "TOMOGRAFIA COMPUTADORIZADA DE COLUNA CERVICAL COM OU SEM CONTRASTE",
      "TOMOGRAFIA COMPUTADORIZADA DE COLUNA LOMBO-SACRA COM OU SEM CONTRASTE",
      "TOMOGRAFIA COMPUTADORIZADA DE COLUNA TORACICA COM OU SEM CONTRASTE",
      "TOMOGRAFIA COMPUTADORIZADA DE FACE / SEIOS DA FACE / ARTICULACOES TEMPORO-MANDIBULARES",
      "TOMOGRAFIA COMPUTADORIZADA DE PELVE / BACIA / ABDOMEN INFERIOR",
      "TOMOGRAFIA COMPUTADORIZADA DE SEGMENTOS APENDICULARES (BRACO, ANTEBRACO, MAO, COXA, PERNA, PE)",
      "TOMOGRAFIA COMPUTADORIZADA DE SELA TURCICA",
      "TOMOGRAFIA COMPUTADORIZADA DE TORAX",
      "TOMOGRAFIA COMPUTADORIZADA DO CRANIO",
      "TOMOGRAFIA COMPUTADORIZADA DO PESCOCO",
      "TRATAMENTO MEDICAMENTOSO DE DOENCA DA RETINA",
      "TRIAGEM - NUCLEO DE FISSURA",
      "ULTRASSONOGRAFIA DE ABDOMEN SUPERIOR",
      "ULTRASSONOGRAFIA DE ABDOMEN TOTAL",
      "ULTRASSONOGRAFIA DE APARELHO URINARIO",
      "ULTRASSONOGRAFIA DE ARTICULACAO",
      "ULTRASSONOGRAFIA DE BOLSA ESCROTAL",
      "ULTRASSONOGRAFIA DE GLOBO OCULAR / ORBITA (MONOCULAR)",
      "ULTRASSONOGRAFIA DE PROSTATA (VIA TRANSRETAL)",
      "ULTRASSONOGRAFIA DE PROSTATA POR VIA ABDOMINAL",
      "ULTRASSONOGRAFIA DE TIREOIDE",
      "ULTRASSONOGRAFIA DE TORAX (EXTRACARDIACA)",
      "ULTRASSONOGRAFIA DOPPLER COLORIDO DE VASOS",
      "ULTRASSONOGRAFIA DOPPLER DE FLUXO OBSTETRICO",
      "ULTRASSONOGRAFIA MAMARIA BILATERAL",
      "ULTRASSONOGRAFIA OBSTETRICA",
      "ULTRASSONOGRAFIA OBSTETRICA C/ DOPPLER COLORIDO E PULSADO",
      "ULTRASSONOGRAFIA PELVICA (GINECOLOGICA)",
      "ULTRASSONOGRAFIA TRANSFONTANELA",
      "ULTRASSONOGRAFIA TRANSVAGINAL",
      "URETROCISTOGRAFIA"
    ]
  },

{
  id: "USG_PORTO_VELHO_HHRO",
  nome: "HOSPITAL DE RETAGUARDA DE RONDONIA - HRRO",
  municipio: "PORTO VELHO",
  tipo: "DIAGNOSTICO (ULTRASSONOGRAFIA)",
  perfil: "Ambulatorial",
  link: "",
  procedimentos: [
    "ULTRASSONOGRAFIA DE APARELHO URINARIO",
    "ULTRASSONOGRAFIA DE ABDOMEN TOTAL",
    "ULTRASSONOGRAFIA TRANSVAGINAL",
    "ULTRASSONOGRAFIA DE TIREOIDE",
    "CONSULTA EM CIRURGIA GERAL",
    "CONSULTA EM ORTOPEDIA"
  ]
},


{
  id: "HOSPITAL_AMOR_AMAZONIA_PVH",
  nome: "HOSPITAL DE AMOR AMAZONIA",
  municipio: "PORTO VELHO",
  tipo: "DIAGNOSTICO / TERAPIA ONCOLOGICA",
  perfil: "Ambulatorial e Hospitalar",
  link: "",
  procedimentos: [
    "ANGIORESSONANCIA CEREBRAL",
    "BRAQUITERAPIA",
    "BRAQUITERAPIA DE ALTA TAXA DE DOSE (POR INSERCAO)",
    "BRAQUITERAPIA GINECOLOGICA",
    "COLONOSCOPIA (COLOSCOPIA)",
    "ESCANOMETRIA",
    "ESOFAGOGASTRODUODENOSCOPIA",

    /* RADIOGRAFIA */
    "RADIOGRAFIA DE ABDOMEN AGUDO (MINIMO DE 3 INCIDENCIAS)",
    "RADIOGRAFIA DE BACIA",
    "RADIOGRAFIA DE BRACO",
    "RADIOGRAFIA DE CALCANEO",
    "RADIOGRAFIA DE COLUNA CERVICAL FUNCIONAL / DINAMICA",
    "RADIOGRAFIA DE COLUNA LOMBO-SACRA",
    "RADIOGRAFIA DE COLUNA LOMBO-SACRA (COM OBLIQUAS)",
    "RADIOGRAFIA DE COLUNA TORACICA (AP + LATERAL)",
    "RADIOGRAFIA DE COLUNA TORACO-LOMBAR",
    "RADIOGRAFIA DE COLUNA TORACO-LOMBAR DINAMICA",
    "RADIOGRAFIA DE COXA",
    "RADIOGRAFIA DE DEDOS DA MAO",
    "RADIOGRAFIA DE ESCAPULA / OMBRO (TRES POSICOES)",
    "RADIOGRAFIA DE JOELHO (AP + LATERAL)",
    "RADIOGRAFIA DE JOELHO OU PATELA (AP + LATERAL + AXIAL)",
    "RADIOGRAFIA DE MAO E PUNHO (PARA DETERMINACAO DE IDADE OSSEA)",
    "RADIOGRAFIA DE PE / DEDOS DO PE",
    "RADIOGRAFIA DE PERNA",
    "RADIOGRAFIA DE PUNHO (AP + LATERAL + OBLIQUA)",
    "RADIOGRAFIA DE SELA TURCICA (PA + LATERAL + BRETTON)",
    "RADIOGRAFIA DE TORAX (PA E PERFIL)",
    "RADIOGRAFIA DE TORAX (PA)",

    /* RADIOTERAPIA */
    "RADIOTERAPIA COM ACELERADOR LINEAR DE FOTONS E ELETRONS (POR CAMPO)",
    "RADIOTERAPIA DE CABECA E PESCOCO",
    "RADIOTERAPIA DE CADEIA LINFATICA",
    "RADIOTERAPIA DE CANCER GINECOLOGICO",
    "RADIOTERAPIA DE DOENCA BENIGNA",
    "RADIOTERAPIA DE LINFOMA E LEUCEMIA",
    "RADIOTERAPIA DE MAMA",
    "RADIOTERAPIA DE METASTASE EM SISTEMA NERVOSO CENTRAL",
    "RADIOTERAPIA DE OSSOS / CARTILAGENS / PARTES MOLES",
    "RADIOTERAPIA DE PELE",
    "RADIOTERAPIA DE PENIS",
    "RADIOTERAPIA DE PLASMOCITOMA / MIELOMA / METASTASES EM OUTRAS LOCALIZACOES",
    "RADIOTERAPIA DE PROSTATA",
    "RADIOTERAPIA DE SISTEMA NERVOSO CENTRAL",
    "RADIOTERAPIA DE TRAQUEIA, BRONQUIO, PULMAO, PLEURA E MEDIASTINO",
    "RADIOTERAPIA DO APARELHO DIGESTIVO",
    "RADIOTERAPIA DO APARELHO URINARIO",
    "RADIOTERAPIA EM CORPO INTEIRO",
    "RADIOTERAPIA ESTEREOTAXICA",

    /* RESSONANCIA */
    "RESSONANCIA MAGNETICA DE ABDOMEN SUPERIOR",
    "RESSONANCIA MAGNETICA DE ARTICULACAO TEMPORO-MANDIBULAR (BILATERAL)",
    "RESSONANCIA MAGNETICA DE BACIA / PELVE / ABDOMEN INFERIOR",
    "RESSONANCIA MAGNETICA DE COLUNA CERVICAL / PESCOCO",
    "RESSONANCIA MAGNETICA DE COLUNA LOMBO-SACRA",
    "RESSONANCIA MAGNETICA DE COLUNA TORACICA",
    "RESSONANCIA MAGNETICA DE CRANIO",
    "RESSONANCIA MAGNETICA DE MAMA BILATERAL PARA AVALIACAO DE POSSIVEIS COMPLICACOES DE IMPLANTE DE PROTESE",
    "RESSONANCIA MAGNETICA DE MEMBRO INFERIOR (UNILATERAL)",
    "RESSONANCIA MAGNETICA DE MEMBRO SUPERIOR (UNILATERAL)",
    "RESSONANCIA MAGNETICA DE SELA TURCICA",
    "RESSONANCIA MAGNETICA DE TORAX",
    "RESSONANCIA MAGNETICA DE VIAS BILIARES / COLANGIORRESSONANCIA",

    "RETOSSIGMOIDOSCOPIA",

    /* TOMOGRAFIA */
    "TOMOGRAFIA COMPUTADORIZADA DE ABDOMEN SUPERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO INFERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO SUPERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA CERVICAL COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA LOMBO-SACRA COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA TORACICA COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE FACE / SEIOS DA FACE / ARTICULACOES TEMPORO-MANDIBULARES",
    "TOMOGRAFIA COMPUTADORIZADA DE PELVE / BACIA / ABDOMEN INFERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE SELA TURCICA",
    "TOMOGRAFIA COMPUTADORIZADA DE TORAX",
    "TOMOGRAFIA COMPUTADORIZADA DO CRANIO",
    "TOMOGRAFIA COMPUTADORIZADA DO PESCOCO",

    "TRAQUEOSCOPIA",

    /* ULTRASSONOGRAFIA */
    "ULTRASSONOGRAFIA DE ABDOMEN SUPERIOR",
    "ULTRASSONOGRAFIA DE ABDOMEN TOTAL",
    "ULTRASSONOGRAFIA DE APARELHO URINARIO",
    "ULTRASSONOGRAFIA DE ARTICULACAO",
    "ULTRASSONOGRAFIA DE BOLSA ESCROTAL",
    "ULTRASSONOGRAFIA DE PROSTATA (VIA TRANSRETAL)",
    "ULTRASSONOGRAFIA DE PROSTATA POR VIA ABDOMINAL",
    "ULTRASSONOGRAFIA DE TIREOIDE",
    "ULTRASSONOGRAFIA DOPPLER COLORIDO DE VASOS",
    "ULTRASSONOGRAFIA OBSTETRICA",
    "ULTRASSONOGRAFIA PELVICA (GINECOLOGICA)",
    "ULTRASSONOGRAFIA TRANSVAGINAL"
  ]
},

{
  id: "CONSULTAS_ONCOLOGIA_HOSPITAL_AMOR_PVH",
  nome: "HOSPITAL DE AMOR AMAZONIA",
  municipio: "PORTO VELHO",
  tipo: "CONSULTAS - ONCOLOGIA",
  perfil: "Ambulatorial",
  link: "",
  procedimentos: [
    "CONSULTA DE PLANEJAMENTO - BRAQUITERAPIA",
    "CONSULTA EM ENFERMAGEM - ONCOLOGIA",
    "CONSULTA EM IODOTERAPIA",
    "CONSULTA EM ONCOLOGIA - RADIOTERAPIA",
    "CONSULTA EM ONCOLOGIA CLINICA - TRIAGEM",
    "CONSULTA EM ONCOLOGIA CLINICA E RADIOTERAPIA"
  ]
},


  /* ===== OFTALMOLOGIA ===== */
  {
    id: "OFT_PORTO_VELHO_CLINICA_SOL",
    nome: "SOL SERVICOS DE OFTALMOLOGIA LTDA (CLINICA SOL)",
    municipio: "PORTO VELHO",
    tipo: "OFTALMOLOGIA",
    perfil: "Ambulatorial",
    link: "",
    procedimentos: [
      "BIOMETRIA ULTRASSONICA (MONOCULAR)",
      "BIOMICROSCOPIA DE FUNDO DE OLHO",
      "CAMPIMETRIA COMPUTADORIZADA OU MANUAL COM GRAFICO",
      "CAPSULECTOMIA POSTERIOR CIRURGICA",
      "CAPSULOTOMIA A YAG LASER",
      "CICLOCRIOCOAGULACAO / DIATERMIA",
      "CONSULTA MEDICA EM OFTALMOLOGIA",
      "CORRECAO CIRURGICA DE ENTROPIO E ECTROPIO",
      "CORRECAO CIRURGICA DE ESTRABISMO (ACIMA DE 2 MUSCULOS)",
      "CORRECAO CIRURGICA DE HERNIA DE IRIS",
      "CORRECAO CIRURGICA DO ESTRABISMO (ATE 2 MUSCULOS)",
      "CURVA DIARIA DE PRESSAO OCULAR CDPO (MINIMO 3 MEDIDAS)",
      "DACRIOCISTORRINOSTOMIA",
      "EVISCERACAO DE GLOBO OCULAR",
      "EXERESE DE CALAZIO E OUTRAS PEQUENAS LESOES DA PALPEBRA E SUPERCILIOS",
      "EXERESE DE TUMOR DE CONJUNTIVA",
      "EXPLANTE DE LENTE INTRA OCULAR",
      "FACECTOMIA COM IMPLANTE DE LENTE INTRA-OCULAR",
      "FACECTOMIA S/ IMPLANTE DE LENTE INTRA-OCULAR",
      "FACOEMULSIFICACAO COM IMPLANTE DE LENTE INTRA-OCULAR DOBRAVEL",
      "FACOEMULSIFICACAO COM IMPLANTE DE LENTE INTRA-OCULAR RIGIDA",
      "FOTOCOAGULACAO A LASER",
      "GONIOSCOPIA",
      "IMPLANTE INTRA-ESTROMAL",
      "IMPLANTE SECUNDARIO DE LENTE INTRA-OCULAR - LIO",
      "INJECAO INTRA-VITREO",
      "INJECAO SUBCONJUTIVAL / SUBTENONIANA",
      "IRIDECTOMIA CIRURGICA",
      "IRIDOTOMIA A LASER",
      "MAPEAMENTO DE RETINA",
      "MICROSCOPIA ESPECULAR DE CORNEA",
      "PAN-FOTOCOAGULACAO DE RETINA A LASER",
      "PAQUIMETRIA ULTRASSONICA",
      "PARACENTESE DE CAMARA ANTERIOR",
      "POTENCIAL DE ACUIDADE VISUAL",
      "PUNCTOPLASTIA",
      "RADIACAO PARA CROSS LINKING CORNEANO",
      "RECOBRIMENTO CONJUNTIVAL",
      "RECONSTITUICAO DE CANAL LACRIMAL",
      "RECONSTITUICAO DE FORNIX CONJUNTIVAL",
      "RECONSTRUCAO DE CAMARA ANTERIOR DO OLHO",
      "REMOCAO DE OLEO DE SILICONE",
      "REPOSICIONAMENTO DE LENTE INTRAOCULAR",
      "RETINOGRAFIA COLORIDA BINOCULAR",
      "RETINOGRAFIA FLUORESCENTE BINOCULAR",
      "RETINOPEXIA COM INTROFLEXAO ESCLERAL",
      "RETIRADA DE CORPO ESTRANHO DA CAMARA ANTERIOR DO OLHO",
      "RETIRADA DE CORPO ESTRANHO DA CORNEA",
      "SIMBLEFAROPLASTIA",
      "SONDAGEM DE VIAS LACRIMAIS",
      "SUBSTITUICAO DE LENTE INTRA-OCULAR",
      "SUTURA DE CONJUNTIVA",
      "SUTURA DE CORNEA",
      "SUTURA DE ESCLERA",
      "SUTURA DE PALPEBRAS",
      "TESTE ORTOPTICO",
      "TOMOGRAFIA DE COERENCIA OPTICA",
      "TONOMETRIA",
      "TOPOGRAFIA COMPUTADORIZADA DE CORNEA",
      "TRABECULECTOMIA",
      "TRATAMENTO CIRURGICO DE BLEFAROCALASE",
      "TRATAMENTO CIRURGICO DE PTERIGIO",
      "TRATAMENTO DE PTOSE PALPEBRAL",
      "TRATAMENTO MEDICAMENTOSO DE DOENCA DA RETINA",
      "ULTRASSONOGRAFIA DE GLOBO OCULAR / ORBITA (MONOCULAR)",
      "VITRECTOMIA ANTERIOR",
      "VITRECTOMIA POSTERIOR",
      "VITRECTOMIA POSTERIOR COM INFUSAO DE PERFLUOCARBONO E ENDOLASER",
      "VITRECTOMIA POSTERIOR COM INFUSAO DE PERFLUOCARBONO/OLEO DE SILICONE/ENDOLASER"
    ]
  },

{
  id: "OFTALMO_ARIQUEMES_NEGR_VENTORIN_MEDICOS_DOS_OLHOS",
  servico: "OFTALMOLOGIA",
  tipo: "OFTALMOLOGIA",
  nome: "NEGREIROS & VENTORIN SERVICOS MEDICOS LTDA (MEDICOS DOS OLHOS)",
  municipio: "ARIQUEMES",
  perfil: "Ambulatorial",
  link: "",
  procedimentos: [
    "BIOMETRIA ULTRASSONICA",
    "BIOMICROSCOPIA DE FUNDO DE OLHO",
    "CAPSULOTOMIA A YAG LASER",
    "CONSULTA EM OFTALMOLOGIA - CATARATA",
    "CONSULTA EM OFTALMOLOGIA - ESTRABISMO",
    "CONSULTA EM OFTALMOLOGIA",
    "CONSULTA EM OFTALMOLOGIA - PTERIGIO",
    "CONSULTA EM OFTALMOLOGIA - RETINA GERAL",
    "CORRECAO CIRURGICA DE ESTRABISMO (ACIMA DE 2 MUSCULOS)",
    "CORRECAO CIRURGICA DO ESTRABISMO (ATE 2 MUSCULOS)",
    "DETERMINACAO DE TEMPO DE SANGRAMENTO - DUKE",
    "EXERESE DE CALAZIO E OUTRAS PEQUENAS LESOES DA PALPEBRA E SUPERCILIOS",
    "EXERESE DE TUMOR DE CONJUNTIVA",
    "FACECTOMIA S/ IMPLANTE DE LENTE INTRAOCULAR",
    "FACOEMULSIFICACAO C/ IMPLANTE DE LENTE INTRAOCULAR DOBRAVEL",
    "FOTOCOAGULACAO A LASER",
    "IMPLANTE SECUNDARIO DE LENTE INTRAOCULAR - LIO",
    "IRIDOTOMIA A LASER",
    "MAPEAMENTO DE RETINA",
    "MICROSCOPIA ESPECULAR DE CORNEA",
    "PAN-FOTOCOAGULACAO DE RETINA A LASER",
    "PAQUIMETRIA ULTRASSONICA",
    "POTENCIAL DE ACUIDADE VISUAL (PAM)",
    "REMOCAO DE OLEO DE SILICONE",
    "REPOSICIONAMENTO DE LENTE INTRAOCULAR",
    "RETINOGRAFIA COLORIDA BINOCULAR",
    "RETINOGRAFIA FLUORESCENTE BINOCULAR (ANGIOGRAFIA)",
    "SUBSTITUICAO DE LENTE INTRAOCULAR",
    "SUTURA DE CORNEA",
    "TOMOGRAFIA DE COERENCIA OPTICA",
    "TONOMETRIA",
    "TRATAMENTO CIRURGICO DE BLEFAROCALASE",
    "TRATAMENTO CIRURGICO DE PTERIGIO",
    "TRATAMENTO MEDICAMENTOSO DA DOENCA DA RETINA",
    "ULTRASSONOGRAFIA DE GLOBO OCULAR / ORBITA",
    "VITRECTOMIA ANTERIOR",
    "VITRECTOMIA POSTERIOR",
    "VITRECTOMIA POSTERIOR COM INFUSAO DE PERFLUOCARBONO E ENDOLASER",
    "VITRECTOMIA POSTERIOR COM INFUSAO DE PERFLUOCARBONO / OLEO DE SILICONE / ENDOLASER"
  ]
},


{
  id: "OFTALMO_CACOAL_NEGR_VENTORIN",
  servico: "OFTALMOLOGIA",
  nome: "NEGREIROS & VENTORIN SERVICOS MEDICOS LTDA",
  municipio: "CACOAL",
 tipo: "OFTALMOLOGIA",
  perfil: "Ambulatorial",
  link: "",
  procedimentos: [
    "BIOMETRIA ULTRASSONICA",
    "BIOMICROSCOPIA DE FUNDO DE OLHO",
    "CONSULTA EM OFTALMOLOGIA",
    "CONSULTA EM OFTALMOLOGIA - CATARATA",
    "CONSULTA EM OFTALMOLOGIA - ESTRABISMO",
    "CONSULTA EM OFTALMOLOGIA - POS OPERATORIO",
    "CONSULTA EM OFTALMOLOGIA - PTERIGIO",
    "CONSULTA EM OFTALMOLOGIA - RETINA GERAL",
    "CORRECAO CIRURGICA DE ESTRABISMO (ACIMA DE 2 MUSCULOS)",
    "CORRECAO CIRURGICA DO ESTRABISMO (ATE 2 MUSCULOS)",
    "DOSAGEM DE GLICOSE",
    "ELETROCARDIOGRAMA - RISCO CIRURGICO",
    "EXERESE DE TUMOR DE CONJUNTIVA",
    "FACECTOMIA S/ IMPLANTE DE LENTE INTRAOCULAR",
    "FACOEMULSIFICACAO C/ IMPLANTE DE LENTE INTRAOCULAR DOBRAVEL",
    "FOTOCOAGULACAO A LASER",
    "GLICEMIA DE JEJUM",
    "HEMOGRAMA COMPLETO",
    "IMPLANTE SECUNDARIO DE LENTE INTRAOCULAR - LIO",
    "MAPEAMENTO DE RETINA",
    "MICROSCOPIA ESPECULAR DE CORNEA",
    "PAN-FOTOCOAGULACAO DE RETINA A LASER",
    "PAQUIMETRIA ULTRASSONICA",
    "POTENCIAL DE ACUIDADE VISUAL (PAM)",
    "RECONSTRUCAO DE CAMARA ANTERIOR DO OLHO",
    "REMOCAO DE OLEO DE SILICONE",
    "REPOSICIONAMENTO DE LENTE INTRAOCULAR",
    "RETINOGRAFIA COLORIDA BINOCULAR",
    "RETINOGRAFIA FLUORESCENTE BINOCULAR (ANGIOGRAFIA)",
    "SUBSTITUICAO DE LENTE INTRAOCULAR",
    "SUTURA DE CORNEA",
    "TOMOGRAFIA DE COERENCIA OPTICA",
    "TONOMETRIA",
    "TRATAMENTO CIRURGICO DE PTERIGIO",
    "TRATAMENTO MEDICAMENTOSO DA DOENCA DA RETINA",
    "ULTRASSONOGRAFIA DE GLOBO OCULAR / ORBITA",
    "VITRECTOMIA ANTERIOR",
    "VITRECTOMIA POSTERIOR",
    "VITRECTOMIA POSTERIOR COM INFUSAO DE PERFLUOCARBONO / OLEO DE SILICONE / ENDOLASER"
  ]
},

{
  id: "OFTALMO_JI_PARANA_IOB_SANTA_CASA",
  servico: "OFTALMOLOGIA",
  nome: "INSTITUTO OFTALMOLOGICO DO BRASIL LTDA (SANTA CASA DE JI PARANA)",
  municipio: "JI-PARANÁ",
 tipo: "OFTALMOLOGIA",
  perfil: "Ambulatorial",
  link: "",
    procedimentos: [
    "ANGIOFLUORESCEINOGRAFIA",
    "MAPEAMENTO DE RETINA",
     "POTENCIAL DE ACUIDADE VISUAL (PAM)",
    "TONOMETRIA",
    "BIOMETRIA ULTRASSONICA",
     "BIOMICROSCOPIA DE FUNDO DE OLHO",
    "CAMPIMETRIA COMPUTADORIZADA",
     "CAPSULOTOMIA A YAG LASER",
"CONSULTA EM OFTALMOLOGIA",
    "CONSULTA EM OFTALMOLOGIA - CATARATA",
    "CONSULTA EM OFTALMOLOGIA - PTERIGIO",
    "CONSULTA EM OFTALMOLOGIA - RETINA - PEDIATRIA",
    "CONSULTA EM OFTALMOLOGIA - RETINA GERAL",
    "CURVA DIARIA DE PRESSAO OCULAR CDPO (MINIMO 3 MEDIDAS)",
    "CURVA DIARIA",
    "FACOEMULSIFICACAO C/ IMPLANTE DE LENTE INTRAOCULAR DOBRAVEL",
    "FUNDOSCOPIA",
    "GONIOSCOPIA",
    "HEMOGRAMA COMPLETO",
    "INJECAO INTRA-VITREO",
    "MICROSCOPIA ESPECULAR DE CORNEA",
   "PAQUIMETRIA ULTRASSONICA",
    "RETINOGRAFIA COLORIDA BINOCULAR",
    "RETINOGRAFIA FLUORESCENTE BINOCULAR (ANGIOGRAFIA)",
    "TOMOGRAFIA DE COERENCIA OPTICA",
    "TOPOGRAFIA COMPUTADORIZADA DE CORNEA",
    "TRATAMENTO CIRURGICO DE PTERIGIO",
    "ULTRASSONOGRAFIA DE GLOBO OCULAR / ORBITA",
    "VITRECTOMIA ANTERIOR",
    "VITRECTOMIA POSTERIOR",
    "VITRECTOMIA POSTERIOR COM INFUSAO DE PERFLUOCARBONO E ENDOLASER",
    "VITRECTOMIA POSTERIOR COM INFUSAO DE PERFLUOCARBONO / OLEO DE SILICONE / ENDOLASER"
  ]
},

  /* ===== DENSITOMETRIA ===== */
  {
    id: "DIA_PORTO_VELHO_DENSYA_DENSITOMETRIA",
    nome: "DENSYA MEDICINA ESPECIALIZADA",
    municipio: "PORTO VELHO",
    tipo: "DIAGNOSTICO (DENSITOMETRIA)",
    perfil: "Ambulatorial",
    link: "",
    procedimentos: [
      "DENSITOMETRIA OSSEA DUO-ENERGETICA DE COLUNA (VERTEBRAS LOMBARES E/OU FEMUR)"
    ]
  },

  /* ===== DIAGNOSTICOS - PORTO VELHO ===== */
 
{
  id: "FUNDACAO_PIO_XII_TC_RM",
  nome: "FUNDACAO PIO XII",
  municipio: "PORTO VELHO",
  tipo: "DIAGNOSTICO (TC E RM)",
  perfil: "Ambulatorial",
  link: "",
  procedimentos: [
    /* RESSONANCIA */
    "RESSONANCIA MAGNETICA DE MEMBRO SUPERIOR (UNILATERAL)",
    "RESSONANCIA MAGNETICA DE BACIA / PELVE / ABDOMEN INFERIOR",
    "RESSONANCIA MAGNETICA DE ABDOMEN SUPERIOR",
    "RESSONANCIA MAGNETICA DE COLUNA LOMBO-SACRA",
    "RESSONANCIA MAGNETICA DE CRANIO",
    "RESSONANCIA MAGNETICA DE COLUNA CERVICAL / PESCOCO",
    "RESSONANCIA MAGNETICA DE MEMBRO INFERIOR (UNILATERAL)",
    "RESSONANCIA MAGNETICA DE TORAX",
    "RESSONANCIA MAGNETICA DE COLUNA TORACICA",
    "RESSONANCIA MAGNETICA DE SELA TURCICA",
    "RESSONANCIA MAGNETICA DE VIAS BILIARES / COLANGIORRESSONANCIA",
    "ANGIORESSONANCIA CEREBRAL",

    /* TOMOGRAFIA */
    "TOMOGRAFIA COMPUTADORIZADA DE TORAX",
    "TOMOGRAFIA COMPUTADORIZADA DE PELVE / BACIA / ABDOMEN INFERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE ABDOMEN SUPERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DO PESCOCO",
    "TOMOGRAFIA COMPUTADORIZADA DE FACE / SEIOS DA FACE / ARTICULACOES TEMPORO-MANDIBULARES",
    "TOMOGRAFIA COMPUTADORIZADA DO CRANIO",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA TORACICA COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA CERVICAL COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE SEGMENTOS APENDICULARES (BRACO, ANTEBRACO, MAO, COXA, PERNA, PE)",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA LOMBO-SACRA COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO SUPERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO INFERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE SELA TURCICA"
  ]
},

 {
    id: "DIA_PORTO_VELHO_ENOCH_RM",
    nome: "ENOCH - UNIDADE DE RADIODIAGNOSTICO E ULTRA-SONOGRAFIA LTDA",
    municipio: "PORTO VELHO",
    tipo: "DIAGNOSTICO (RM)",
    perfil: "Ambulatorial",
    link: "",
    procedimentos: [
      "RESSONANCIA MAGNETICA DE COLUNA CERVICAL / PESCOCO",
      "RESSONANCIA MAGNETICA DE COLUNA LOMBO-SACRA",
      "RESSONANCIA MAGNETICA DE MEMBRO INFERIOR (UNILATERAL)",
      "ANGIORESSONANCIA CEREBRAL",
      "RESSONANCIA MAGNETICA DE COLUNA TORACICA",
      "RESSONANCIA MAGNETICA DE BACIA / PELVE / ABDOMEN INFERIOR",
      "RESSONANCIA MAGNETICA DE CRANIO",
      "RESSONANCIA MAGNETICA DE MEMBRO SUPERIOR (UNILATERAL)",
      "RESSONANCIA MAGNETICA DE VIAS BILIARES / COLANGIORRESSONANCIA",
      "RESSONANCIA MAGNETICA DE ABDOMEN SUPERIOR",
      "RESSONANCIA MAGNETICA DE SELA TURCICA",
      "RESSONANCIA MAGNETICA DE TORAX",
      "RESSONANCIA MAGNETICA DE CORACAO / AORTA C/ CINE",
      "RESSONANCIA MAGNETICA DE ARTICULACAO TEMPORO-MANDIBULAR (BILATERAL)"
    ]
  },
{
  id: "DIA_NEUROPH_EEG",
  nome: "NEUROPH",
  municipio: "PORTO VELHO", // INFORMAR MUNICIPIO
  tipo: "ELETROENCEFALOGRAFIA (EEG)",
  perfil: "Ambulatorial",
  link: "",
  procedimentos: [
    "ELETROENCEFALOGRAMA EM VIGILIA E SONO ESPONTANEO COM OU SEM FOTOESTIMULO (EEG)",
    "ELETROENCEFALOGRAMA QUANTITATIVO COM MAPEAMENTO (EEG)",
    "ELETROENCEFALOGRAFIA EM VIGILIA COM OU SEM FOTOESTIMULO",
    "ELETROENCEFALOGRAMA EM SONO INDUZIDO COM OU SEM MEDICAMENTO (EEG)"
  ]
},

  {
    id: "DIA_PORTO_VELHO_ALPHACLIN_RM",
    nome: "ALPHACLIN - UNIDADE DE ULTRASSONOGRAFIA DE RONDONIA LTDA",
    municipio: "PORTO VELHO",
    tipo: "DIAGNOSTICO (RM)",
    perfil: "Ambulatorial",
    link: "",
    procedimentos: [
      "RESSONANCIA MAGNETICA DE MEMBRO SUPERIOR (UNILATERAL)",
      "RESSONANCIA MAGNETICA DE COLUNA LOMBO-SACRA",
      "ANGIORESSONANCIA CEREBRAL",
      "RESSONANCIA MAGNETICA DE COLUNA CERVICAL / PESCOCO",
      "RESSONANCIA MAGNETICA DE COLUNA TORACICA",
      "RESSONANCIA MAGNETICA DE BACIA / PELVE / ABDOMEN INFERIOR",
      "RESSONANCIA MAGNETICA DE MEMBRO INFERIOR (UNILATERAL)",
      "RESSONANCIA MAGNETICA DE ABDOMEN SUPERIOR",
      "RESSONANCIA MAGNETICA DE CRANIO",
      "RESSONANCIA MAGNETICA DE ARTICULACAO TEMPORO-MANDIBULAR (BILATERAL)",
      "RESSONANCIA MAGNETICA DE TORAX",
      "RESSONANCIA MAGNETICA DE VIAS BILIARES / COLANGIORRESSONANCIA",
      "RESSONANCIA MAGNETICA DE SELA TURCICA"
    ]
  },

  /* ===== DIAGNOSTICOS - ARIQUEMES ===== */

{
  id: "DIA_ARIQUEMES_IMEDI_RM",
  nome: "IMEDI",
  municipio: "ARIQUEMES",
  tipo: "DIAGNOSTICO (RM)",
  perfil: "Ambulatorial",
  link: "",
  procedimentos: [
    "RESSONANCIA MAGNETICA DE MEMBRO INFERIOR (UNILATERAL)",
    "ANGIORESSONANCIA CEREBRAL",
    "RESSONANCIA MAGNETICA DE BACIA / PELVE / ABDOMEN INFERIOR",
    "RESSONANCIA MAGNETICA DE TORAX",
    "RESSONANCIA MAGNETICA DE CRANIO",
    "RESSONANCIA MAGNETICA DE COLUNA LOMBO-SACRA",
    "RESSONANCIA MAGNETICA DE COLUNA TORACICA",
    "RESSONANCIA MAGNETICA DE COLUNA CERVICAL / PESCOCO",
    "RESSONANCIA MAGNETICA DE MEMBRO SUPERIOR (UNILATERAL)",
    "RESSONANCIA MAGNETICA DE ABDOMEN SUPERIOR",
    "RESSONANCIA MAGNETICA DE VIAS BILIARES / COLANGIORRESSONANCIA",
    "RESSONANCIA MAGNETICA DE CORACAO / AORTA C/ CINE",
    "RESSONANCIA MAGNETICA DE SELA TURCICA"
  ]
},

  /* ===== DIAGNOSTICOS - JARU ===== */

{
  id: "DIA_JARU_GASTROIMAGEM_TC_RM",
  nome: "GASTROIMAGEM LTDA",
  municipio: "JARU",
  tipo: "DIAGNOSTICO (TC/RM)",
  perfil: "Ambulatorial",
  link: "",
  procedimentos: [
    /* ===== RESSONANCIA MAGNETICA ===== */
    "RESSONANCIA MAGNETICA DE MEMBRO INFERIOR (UNILATERAL)",
    "RESSONANCIA MAGNETICA DE COLUNA LOMBO-SACRA",
    "RESSONANCIA MAGNETICA DE COLUNA CERVICAL / PESCOCO",
    "RESSONANCIA MAGNETICA DE COLUNA TORACICA",
    "RESSONANCIA MAGNETICA DE BACIA / PELVE / ABDOMEN INFERIOR",
    "RESSONANCIA MAGNETICA DE MEMBRO SUPERIOR (UNILATERAL)",
    "RESSONANCIA MAGNETICA DE CRANIO",
    "RESSONANCIA MAGNETICA DE ABDOMEN SUPERIOR",
    "RESSONANCIA MAGNETICA DE TORAX",
    "RESSONANCIA MAGNETICA DE ARTICULACAO TEMPORO-MANDIBULAR (BILATERAL)",
    "RESSONANCIA MAGNETICA DE SELA TURCICA",

    /* ===== TOMOGRAFIA COMPUTADORIZADA ===== */
    "TOMOGRAFIA COMPUTADORIZADA DE TORAX",
    "TOMOGRAFIA COMPUTADORIZADA DE ABDOMEN SUPERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DO CRANIO",
    "TOMOGRAFIA COMPUTADORIZADA DE PELVE / BACIA / ABDOMEN INFERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA LOMBO-SACRA COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO INFERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE FACE / SEIOS DA FACE / ARTICULACOES TEMPORO-MANDIBULARES",
    "TOMOGRAFIA COMPUTADORIZADA DE SELA TURCICA",
    "TOMOGRAFIA COMPUTADORIZADA DO PESCOCO",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA TORACICA COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA CERVICAL COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE SEGMENTOS APENDICULARES (BRACO, ANTEBRACO, MAO, COXA, PERNA, PE)",
    "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO SUPERIOR"
  ]
},


  /* ===== DIAGNOSTICOS - CEREJEIRAS ===== */
{
  id: "DIA_MEGA_IMAGEM_TC_RM",
  nome: "MEGA IMAGEM",
  municipio: "CEREJEIRAS", // INFORMAR MUNICIPIO
  tipo: "DIAGNOSTICO (TC/RM)",
  perfil: "Ambulatorial",
  link: "",
  procedimentos: [
    /* ===== TOMOGRAFIA ===== */
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA LOMBO-SACRA COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE ABDOMEN SUPERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE TORAX",
    "TOMOGRAFIA COMPUTADORIZADA DO PESCOCO",
    "TOMOGRAFIA COMPUTADORIZADA DE PELVE / BACIA / ABDOMEN INFERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DO CRANIO",
    "TOMOGRAFIA COMPUTADORIZADA DE FACE / SEIOS DA FACE / ARTICULACOES TEMPORO-MANDIBULARES",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA CERVICAL COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA TORACICA COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO INFERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO SUPERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE SELA TURCICA",
    "TOMOGRAFIA COMPUTADORIZADA DE SEGMENTOS APENDICULARES (BRACO, ANTEBRACO, MAO, COXA, PERNA, PE)",

    /* ===== RESSONANCIA ===== */
    "RESSONANCIA MAGNETICA DE BACIA / PELVE / ABDOMEN INFERIOR",
    "RESSONANCIA MAGNETICA DE COLUNA LOMBO-SACRA",
    "RESSONANCIA MAGNETICA DE MEMBRO INFERIOR (UNILATERAL)",
    "RESSONANCIA MAGNETICA DE COLUNA TORACICA",
    "RESSONANCIA MAGNETICA DE TORAX",
    "RESSONANCIA MAGNETICA DE COLUNA CERVICAL / PESCOCO",
    "RESSONANCIA MAGNETICA DE MEMBRO SUPERIOR (UNILATERAL)",
    "RESSONANCIA MAGNETICA DE ABDOMEN SUPERIOR",
    "RESSONANCIA MAGNETICA DE CRANIO",
    "RESSONANCIA MAGNETICA DE SELA TURCICA",
    "RESSONANCIA MAGNETICA DE ARTICULACAO TEMPORO-MANDIBULAR (BILATERAL)",
    "RESSONANCIA MAGNETICA DE VIAS BILIARES / COLANGIORRESSONANCIA"
  ]
},  



{
    id: "DIA_CEREJEIRAS_CLIMEDI_TC",
    nome: "CLIMEDI LTDA",
    municipio: "CEREJEIRAS",
    tipo: "DIAGNOSTICO (TC)",
    perfil: "Ambulatorial",
    link: "",
    procedimentos: [
      "TOMOGRAFIA COMPUTADORIZADA DE PELVE / BACIA / ABDOMEN INFERIOR",
      "TOMOGRAFIA COMPUTADORIZADA DE TORAX",
      "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO INFERIOR",
      "TOMOGRAFIA COMPUTADORIZADA DE COLUNA TORACICA COM OU SEM CONTRASTE",
      "TOMOGRAFIA COMPUTADORIZADA DO CRANIO",
      "TOMOGRAFIA COMPUTADORIZADA DE SELA TURCICA",
      "TOMOGRAFIA COMPUTADORIZADA DE FACE / SEIOS DA FACE / ARTICULACOES TEMPORO-MANDIBULARES",
      "TOMOGRAFIA COMPUTADORIZADA DE ABDOMEN SUPERIOR",
      "TOMOGRAFIA COMPUTADORIZADA DE COLUNA LOMBO-SACRA COM OU SEM CONTRASTE",
      "TOMOGRAFIA COMPUTADORIZADA DE COLUNA CERVICAL COM OU SEM CONTRASTE",
      "TOMOGRAFIA COMPUTADORIZADA DO PESCOCO",
      "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO SUPERIOR",
      "TOMOGRAFIA COMPUTADORIZADA DE SEGMENTOS APENDICULARES (BRACO, ANTEBRACO, MAO, COXA, PERNA, PE)"
    ]
  },

  /* ===== CACOAL ===== */
  {
    id: "CIR_CACOAL_HOSPITAL_ACIDENTADOS",
    nome: "AZEVEDO & AZEVEDO LTDA (HOSPITAL DOS ACIDENTADOS)",
    municipio: "CACOAL",
    tipo: "CIRURGIAS",
    link: "",
    procedimentos: ["ARTROPLASTIA DE QUADRIL (NAO CONVENCIONAL)"]
  },




  /* ===== DIAGNOSTICO CACOAL ===== */


{
  id: "DIA_CACOAL_NOVA_IMAGEM_TC",
  nome: "NOVA IMAGEM DIAGNOSTICO",
  municipio: "CACOAL",
  tipo: "DIAGNOSTICO (TC)",
  perfil: "Ambulatorial",
  link: "",
  procedimentos: [
    "TOMOGRAFIA COMPUTADORIZADA DE PELVE / BACIA / ABDOMEN INFERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE ABDOMEN SUPERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DO PESCOCO",
    "TOMOGRAFIA COMPUTADORIZADA DE FACE / SEIOS DA FACE / ARTICULACOES TEMPORO-MANDIBULARES",
    "TOMOGRAFIA COMPUTADORIZADA DO CRANIO",
    "TOMOGRAFIA COMPUTADORIZADA DE TORAX",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA CERVICAL COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA LOMBO-SACRA COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO INFERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA TORACICA COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE SEGMENTOS APENDICULARES (BRACO, ANTEBRACO, MAO, COXA, PERNA, PE)",
    "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO SUPERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE SELA TURCICA"
  ]
},


  {
    id: "DIA_CACOAL_ANGA_TC_RM",
    nome: "ANGA - CENTRO DE MEDICINA DIAGNOSTICA LTDA",
    municipio: "CACOAL",
    tipo: "DIAGNOSTICO (TC/RM)",
    perfil: "Ambulatorial",
    link: "",
    procedimentos: [
      /* TC */
      "TOMOGRAFIA COMPUTADORIZADA DE TORAX",
      "TOMOGRAFIA COMPUTADORIZADA DE COLUNA LOMBO-SACRA COM OU SEM CONTRASTE",
      "TOMOGRAFIA COMPUTADORIZADA DO CRANIO",
      "TOMOGRAFIA COMPUTADORIZADA DE PELVE / BACIA / ABDOMEN INFERIOR",
      "TOMOGRAFIA COMPUTADORIZADA DE ABDOMEN SUPERIOR",
      "TOMOGRAFIA COMPUTADORIZADA DE FACE / SEIOS DA FACE / ARTICULACOES TEMPORO-MANDIBULARES",
      "TOMOGRAFIA COMPUTADORIZADA DO PESCOCO",
      "TOMOGRAFIA COMPUTADORIZADA DE COLUNA CERVICAL COM OU SEM CONTRASTE",
      "TOMOGRAFIA COMPUTADORIZADA DE COLUNA TORACICA COM OU SEM CONTRASTE",
      "TOMOGRAFIA COMPUTADORIZADA DE SEGMENTOS APENDICULARES (BRACO, ANTEBRACO, MAO, COXA, PERNA, PE)",
      "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO SUPERIOR",
      "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO INFERIOR",
      "TOMOGRAFIA COMPUTADORIZADA DE SELA TURCICA",

      /* RM */
      "RESSONANCIA MAGNETICA DE COLUNA CERVICAL / PESCOCO",
      "RESSONANCIA MAGNETICA DE COLUNA LOMBO-SACRA",
      "RESSONANCIA MAGNETICA DE MEMBRO SUPERIOR (UNILATERAL)",
      "RESSONANCIA MAGNETICA DE CRANIO",
      "RESSONANCIA MAGNETICA DE MEMBRO INFERIOR (UNILATERAL)",
      "RESSONANCIA MAGNETICA DE BACIA / PELVE / ABDOMEN INFERIOR",
      "RESSONANCIA MAGNETICA DE COLUNA TORACICA",
      "RESSONANCIA MAGNETICA DE ABDOMEN SUPERIOR",
      "RESSONANCIA MAGNETICA DE TORAX",
      "RESSONANCIA MAGNETICA DE ARTICULACAO TEMPORO-MANDIBULAR (BILATERAL)",
      "RESSONANCIA MAGNETICA DE SELA TURCICA",
      "ANGIORESSONANCIA CEREBRAL",
      "RESSONANCIA MAGNETICA DE VIAS BILIARES / COLANGIORRESSONANCIA"
    ]
  },


 
{
  id: "DIA_PORTO_VELHO_RADIOIMAGEM_RM",
  nome: "CENTRO DE DIAGNOSTICO RADIOIMAGEM LTDA",
  municipio: "PORTO VELHO",
  tipo: "DIAGNOSTICO (TC/RM)",
  perfil: "Ambulatorial",
  link: "",
  procedimentos: [
    /* RESSONANCIA */
    "RESSONANCIA MAGNETICA DE COLUNA LOMBO-SACRA",
    "RESSONANCIA MAGNETICA DE COLUNA CERVICAL / PESCOCO",
    "RESSONANCIA MAGNETICA DE BACIA / PELVE / ABDOMEN INFERIOR",
    "RESSONANCIA MAGNETICA DE MEMBRO INFERIOR (UNILATERAL)",
    "RESSONANCIA MAGNETICA DE COLUNA TORACICA",
    "RESSONANCIA MAGNETICA DE MEMBRO SUPERIOR (UNILATERAL)",
    "RESSONANCIA MAGNETICA DE VIAS BILIARES / COLANGIORRESSONANCIA",
    "RESSONANCIA MAGNETICA DE CRANIO",
    "RESSONANCIA MAGNETICA DE ABDOMEN SUPERIOR",
    "RESSONANCIA MAGNETICA DE TORAX",
    "RESSONANCIA MAGNETICA DE ARTICULACAO TEMPORO-MANDIBULAR (BILATERAL)",
    "RESSONANCIA MAGNETICA DE CORACAO / AORTA COM CINE",
    "RESSONANCIA MAGNETICA DE SELA TURCICA",
    "RESSONANCIA MAGNETICA MULTIPARAMETRICA DA PROSTATA",
    "ANGIORESSONANCIA CEREBRAL",

    /* TOMOGRAFIA */
    "TOMOGRAFIA COMPUTADORIZADA DO CRANIO",
    "TOMOGRAFIA COMPUTADORIZADA DE PELVE / BACIA / ABDOMEN INFERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE ABDOMEN SUPERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE TORAX",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA CERVICAL COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE FACE / SEIOS DA FACE / ARTICULACOES TEMPORO-MANDIBULARES",
    "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO SUPERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA TORACICA COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DO PESCOCO",
    "TOMOGRAFIA COMPUTADORIZADA DE COLUNA LOMBO-SACRA COM OU SEM CONTRASTE",
    "TOMOGRAFIA COMPUTADORIZADA DE ARTICULACOES DE MEMBRO INFERIOR",
    "TOMOGRAFIA COMPUTADORIZADA DE SEGMENTOS APENDICULARES (BRACO, ANTEBRACO, MAO, COXA, PERNA, PE)",
    "TOMOGRAFIA COMPUTADORIZADA DE SELA TURCICA"
  ]
},


/* ===== DIAGNOSTICO - PIMENTA BUENO ===== */



{
  id: "DIA_PIMENTA_BUENO_RD",
  nome: "R D SERVICOS MEDICOS LTDA",
  municipio: "PIMENTA BUENO",
  tipo: "DIAGNOSTICO: ELETROENCEFALO E ELETRONEURO",
  perfil: "Ambulatorial",
  link: "",
  procedimentos: [
    "ELETROENCEFALOGRAMA EM SONO INDUZIDO COM OU SEM MEDICAMENTO (EEG)",
    "ELETROENCEFALOGRAMA EM VIGILIA E SONO ESPONTANEO COM OU SEM FOTOESTIMULO (EEG)",
    "ELETRONEUROMIOGRAMA (ENMG)",
    "ELETROENCEFALOGRAFIA EM VIGILIA COM OU SEM FOTOESTIMULO",
    "ELETROENCEFALOGRAMA QUANTITATIVO COM MAPEAMENTO (EEG)"
  ]
}



];

/* =========================
   HELPERS
   ========================= */
function norm(s){
  return (s||"").toString()
    .normalize("NFD").replace(/[\u0300-\u036f]/g,"")
    .replace(/\s+/g," ")
    .trim()
    .toUpperCase();
}
function uniq(arr){ return Array.from(new Set(arr)); }
function sortPT(arr){ return arr.slice().sort((a,b)=>a.localeCompare(b,"pt-BR")); }
function escapeHtml(s){
  return String(s||"")
    .replaceAll("&","&amp;")
    .replaceAll("<","&lt;")
    .replaceAll(">","&gt;")
    .replaceAll('"',"&quot;")
    .replaceAll("'","&#039;");
}
function highlight(text, query){
  const t = String(text||"");
  const q = String(query||"").trim();
  if (!q) return escapeHtml(t);

  // highlight sem regex “perigoso”: faz por índice (case-insensitive, com normalização simples)
  const tn = norm(t);
  const qn = norm(q);

  const idx = tn.indexOf(qn);
  if (idx < 0) return escapeHtml(t);

  // como norm pode alterar tamanho, a marcação é aproximada pelo texto original (bom o suficiente p/ UI)
  // estratégia: busca case-insensitive no texto original sem acento removido
  const idx2 = t.toUpperCase().indexOf(q.toUpperCase());
  if (idx2 < 0) return escapeHtml(t);

  const a = t.slice(0, idx2);
  const b = t.slice(idx2, idx2 + q.length);
  const c = t.slice(idx2 + q.length);
  return `${escapeHtml(a)}<span class="acMark">${escapeHtml(b)}</span>${escapeHtml(c)}`;
}

/* =========================
   STATE
   ========================= */
const state = {
  mode: "servicos",
  municipio: "",
  tipoProc: "",
  procedimento: "",
  unidade: "",
  perfil: ""
};

/* =========================
   GETTERS
   ========================= */
function allMunicipios(){
  const a = Object.keys(rede);
  const b = providers.map(p => p.municipio);
  return sortPT(uniq([...a, ...b]));
}
function allTipoProcedimentoFromRede(){
  const s = [];
  Object.values(rede).forEach(lista => lista.forEach(item => s.push(item.servico)));
  return sortPT(uniq(s));
}
function allUnits(){
  return sortPT(uniq(providers.map(p => p.nome)));
}
function allProcedures(){
  const procs = [];
  providers.forEach(p => (p.procedimentos||[]).forEach(x => procs.push(x)));
  return sortPT(uniq(procs));
}

/* ---- SERVICOS ---- */
function getItemsServicosFiltered(customState=null){
  const s = customState || state;
  let municipios = Object.keys(rede);

  if (s.municipio){
    municipios = municipios.filter(m => norm(m) === norm(s.municipio));
  }

  let items = [];
  municipios.forEach(m => (rede[m]||[]).forEach(it => items.push({ municipio: m, ...it })));

  if (s.tipoProc){
    const tp = norm(s.tipoProc);
    items = items.filter(it => norm(it.servico) === tp);
  }
  return items;
}

/* ---- PROCEDIMENTOS ---- */
function getProvidersFiltered(customState=null){
  const s = customState || state;
  let list = providers.slice();

  if (s.perfil){
    list = list.filter(p => norm(p.perfil||"") === norm(s.perfil));
  }
  if (s.unidade){
    list = list.filter(p => norm(p.nome) === norm(s.unidade));
  }
  if (s.procedimento){
    const proc = norm(s.procedimento);
    list = list.filter(p => (p.procedimentos||[]).some(x => norm(x) === proc));
  }
  return list;
}

/* =========================
   UI POPULATE
   ========================= */
let PROC_ALL = [];
function fillUI(){
  const munSel = document.getElementById("munSel");
  munSel.innerHTML = '<option value="">Todos</option>';
  allMunicipios().forEach(m => {
    const opt = document.createElement("option");
    opt.value = m;
    opt.textContent = m;
    munSel.appendChild(opt);
  });

  const tipoProcSel = document.getElementById("tipoProcSel");
  tipoProcSel.innerHTML = '<option value="">Todos</option>';
  allTipoProcedimentoFromRede().forEach(s => {
    const opt = document.createElement("option");
    opt.value = s;
    opt.textContent = s;
    tipoProcSel.appendChild(opt);
  });

  const unitSel = document.getElementById("unitSel");
  unitSel.innerHTML = '<option value="">Todas</option>';
  allUnits().forEach(u => {
    const opt = document.createElement("option");
    opt.value = u;
    opt.textContent = u;
    unitSel.appendChild(opt);
  });

  PROC_ALL = allProcedures();
}

/* =========================
   MODO UI
   ========================= */
function showProcPanel(show){
  document.getElementById("procPanel").classList.toggle("hidden", !show);
}
function setModeUI(){
  const isServ = state.mode === "servicos";

  document.getElementById("munControl").classList.toggle("hidden", !isServ);
  document.getElementById("tipoProcControl").classList.toggle("hidden", !isServ);

  document.getElementById("unitControl").classList.toggle("hidden", isServ);
  document.getElementById("perfilControl").classList.toggle("hidden", isServ);
  document.getElementById("procControl").classList.toggle("hidden", isServ);

  showProcPanel(isServ);

  if (isServ){
    state.unidade = "";
    state.perfil = "";
    state.procedimento = "";
    document.getElementById("unitSel").value = "";
    document.getElementById("perfilSel").value = "";
    document.getElementById("procInput").value = "";
    closeSuggest();
  } else {
    state.municipio = "";
    state.tipoProc = "";
    document.getElementById("munSel").value = "";
    document.getElementById("tipoProcSel").value = "";
  }
}

function chipText(){
  const parts = [];
  parts.push(state.mode === "servicos" ? "MODO: SERVICOS" : "MODO: PROCEDIMENTOS");
  if (state.mode === "servicos"){
    if (state.municipio) parts.push("MUNICIPIO: " + state.municipio);
    if (state.tipoProc) parts.push("TIPO: " + state.tipoProc);
  } else {
    if (state.perfil) parts.push("PERFIL: " + state.perfil);
    if (state.unidade) parts.push("UNIDADE: " + state.unidade);
    if (state.procedimento) parts.push("PROCEDIMENTO: " + state.procedimento);
  }
  return parts.join(" | ") || "SEM FILTRO";
}

/* =========================
   DETALHE (modo procedimentos)
   ========================= */
function renderDetailEmpty(){
  document.getElementById("detTitle").textContent = "Detalhe";
  document.getElementById("detMeta").textContent = "Selecione uma unidade (modo Procedimentos) ou clique no municipio no mapa.";
  document.getElementById("detail").innerHTML = "<div class='muted'>Sem detalhe selecionado.</div>";
}

function renderProceduresDetail(provider){
  document.getElementById("detTitle").textContent = "Unidade - " + provider.nome;
  document.getElementById("detMeta").textContent =
    provider.municipio + " | " + (provider.tipo||"-") + " | " + (provider.perfil||"-") + " | " + (provider.procedimentos||[]).length + " procedimento(s)";

  const detail = document.getElementById("detail");
  detail.innerHTML = "";

  const link = provider.link && String(provider.link).trim() && provider.link !== "URL_DA_CARTA_DE_SERVICOS" ? provider.link : "";

  const wrap = document.createElement("div");
  wrap.className = "tableWrap";
  wrap.innerHTML = `
    <table>
      <thead>
        <tr>
          <th>Procedimento</th>
          <th style="width:240px;">Identificação</th>
        </tr>
      </thead>
      <tbody>
        ${
          (provider.procedimentos||[]).length
            ? provider.procedimentos
                .slice()
                .sort((a,b)=>a.localeCompare(b,"pt-BR"))
                .map(proc => `
                  <tr>
                    <td><b>${escapeHtml(proc)}</b></td>
                    <td><span class="badge"><strong>${escapeHtml(provider.perfil || "-")}</strong> • ${escapeHtml(provider.municipio)}</span></td>
                  </tr>
                `).join("")
            : `<tr><td colspan="2" class="muted">Sem rol cadastrado.</td></tr>`
        }
      </tbody>
    </table>
  `;
  detail.appendChild(wrap);

  const linkBox = document.createElement("div");
  linkBox.className = "box";
  linkBox.style.marginTop = "10px";
  linkBox.innerHTML = `
    <h3>Carta de Serviços</h3>
    <div class="meta">${link ? "Clique para abrir." : "Sem link cadastrado (ou pendente)."}</div>
    <div>${link ? `<a class="linkBtn" href="${link}" target="_blank" rel="noopener">Abrir</a>` : `<span class="badge"><strong>Sem</strong> link</span>`}</div>
  `;
  detail.appendChild(linkBox);
}

/* =========================
   Painel destacado: Procedimentos no modo SERVIÇOS
   ========================= */
let PROC_PANEL_DATA = { prestador:"", providers:[], procedimentos:[], index:new Map() };
let PROC_SELECTED = "";

function buildProcPanelForPrestador(prestadorName){
  const target = norm(prestadorName);
  const matches = providers.filter(p => norm(p.nome) === target);

  PROC_PANEL_DATA.prestador = prestadorName;
  PROC_PANEL_DATA.providers = matches;

  const all = [];
  const idx = new Map();
  matches.forEach(p => {
    (p.procedimentos||[]).forEach(proc => {
      all.push(proc);
      const k = norm(proc);
      if (!idx.has(k)) idx.set(k, []);
      idx.get(k).push(p);
    });
  });

  const unique = sortPT(uniq(all));
  PROC_PANEL_DATA.procedimentos = unique;
  PROC_PANEL_DATA.index = idx;
  PROC_SELECTED = "";

  document.getElementById("procPanelTitle").textContent = `Procedimentos — ${prestadorName}`;
  document.getElementById("procPanelMeta").textContent =
    matches.length
      ? `${matches.length} cadastro(s) em providers | Clique em um procedimento para ver detalhes abaixo.`
      : "Sem rol cadastrado em providers para esse prestador (verifique o nome).";

  document.getElementById("procCountChip").textContent = `${unique.length} itens`;
  document.getElementById("procListFilter").value = "";

  renderProcPanelList();
  renderProcPanelDetail(null);
  showProcPanel(true);
}

function renderProcPanelList(){
  const listEl = document.getElementById("procList");
  listEl.innerHTML = "";

  const filter = norm(document.getElementById("procListFilter").value || "");
  const base = PROC_PANEL_DATA.procedimentos || [];
  const view = filter ? base.filter(p => norm(p).includes(filter)) : base;

  document.getElementById("procCountChip").textContent = `${view.length} itens`;

  if (!view.length){
    listEl.innerHTML = `<div class="muted">Nenhum procedimento para esse filtro.</div>`;
    return;
  }

  view.forEach(proc => {
    const item = document.createElement("div");
    item.className = "procItem" + (norm(proc) === norm(PROC_SELECTED) ? " active" : "");
    item.innerHTML = `
      <div>
        <b>${escapeHtml(proc)}</b>
        <small>${escapeHtml(PROC_PANEL_DATA.prestador)}</small>
      </div>
      <div><span class="chip">Detalhar</span></div>
    `;
    item.addEventListener("click", () => {
      PROC_SELECTED = proc;
      renderProcPanelList();
      renderProcPanelDetail(proc);
    });
    listEl.appendChild(item);
  });
}

function renderProcPanelDetail(proc){
  const title = document.getElementById("procDetailTitle");
  const chip  = document.getElementById("procDetailChip");
  const meta  = document.getElementById("procDetailMeta");
  const body  = document.getElementById("procDetailBody");

  if (!proc){
    title.textContent = "Detalhe";
    chip.textContent = "—";
    meta.textContent = "Selecione um procedimento na lista acima.";
    body.innerHTML = `<div class="muted">Nenhum procedimento selecionado.</div>`;
    return;
  }

  title.textContent = proc;
  chip.textContent = "Selecionado";
  const key = norm(proc);
  const provs = PROC_PANEL_DATA.index.get(key) || [];
  meta.textContent = `${provs.length} cadastro(s) onde aparece este procedimento (município/tipo/perfil).`;

  body.innerHTML = "";
  if (!provs.length){
    body.innerHTML = `<div class="muted">Sem detalhes disponíveis.</div>`;
    return;
  }

  const wrap = document.createElement("div");
  wrap.className = "tableWrap";
  wrap.innerHTML = `
    <table>
      <thead>
        <tr>
          <th>Município</th>
          <th>Tipo</th>
          <th style="width:160px;">Perfil</th>
          <th style="width:160px;">Carta</th>
        </tr>
      </thead>
      <tbody>
        ${provs
          .slice()
          .sort((a,b)=> (a.municipio||"").localeCompare(b.municipio||"","pt-BR"))
          .map(p => `
            <tr>
              <td><b>${escapeHtml(p.municipio || "-")}</b></td>
              <td>${escapeHtml(p.tipo || "-")}</td>
              <td><span class="badge"><strong>${escapeHtml(p.perfil || "-")}</strong></span></td>
              <td>
                ${
                  p.link && String(p.link).trim() && p.link !== "URL_DA_CARTA_DE_SERVICOS"
                    ? `<a class="linkBtn" href="${p.link}" target="_blank" rel="noopener">Abrir</a>`
                    : `<span class="badge"><strong>Sem</strong> link</span>`
                }
              </td>
            </tr>
          `).join("")}
      </tbody>
    </table>
  `;
  body.appendChild(wrap);
}
document.getElementById("procListFilter").addEventListener("input", renderProcPanelList);

/* =========================
   AUTOCOMPLETE PROFISSIONAL (custom)
   ========================= */
const procInput = document.getElementById("procInput");
const procSuggest = document.getElementById("procSuggest");

let SUGG_VIEW = [];
let SUGG_ACTIVE = -1;

function closeSuggest(){
  procSuggest.classList.remove("show");
  procSuggest.innerHTML = "";
  SUGG_VIEW = [];
  SUGG_ACTIVE = -1;
}
function openSuggest(){
  if (!procSuggest.classList.contains("show")) procSuggest.classList.add("show");
}
function setActive(idx){
  SUGG_ACTIVE = idx;
  [...procSuggest.querySelectorAll(".acItem")].forEach((el,i)=>{
    el.classList.toggle("active", i===idx);
  });
  const activeEl = procSuggest.querySelector(".acItem.active");
  if (activeEl){
    const r = activeEl.getBoundingClientRect();
    const pr = procSuggest.getBoundingClientRect();
    if (r.top < pr.top + 40) procSuggest.scrollTop -= (pr.top + 40 - r.top);
    if (r.bottom > pr.bottom - 10) procSuggest.scrollTop += (r.bottom - (pr.bottom - 10));
  }
}

function buildProcIndex(){
  // dica: aqui dá para evoluir depois com SIGTAP/códigos
  return PROC_ALL.slice();
}

function renderSuggest(query){
  const q = String(query||"").trim();
  const qn = norm(q);

  if (!q){
    closeSuggest();
    return;
  }

  const base = buildProcIndex();
  let view = base.filter(p => norm(p).includes(qn));

  // ordenação: prioriza início
  view.sort((a,b)=>{
    const an = norm(a), bn = norm(b);
    const ai = an.indexOf(qn), bi = bn.indexOf(qn);
    if (ai !== bi) return ai - bi;
    return a.localeCompare(b, "pt-BR");
  });

  const LIMIT = 120;
  view = view.slice(0, LIMIT);

  SUGG_VIEW = view;
  SUGG_ACTIVE = view.length ? 0 : -1;

  if (!view.length){
    procSuggest.innerHTML = `
      <div class="acHeader">
        <span class="chip">0 resultados</span>
        <span class="muted">Refine o termo</span>
      </div>
      <div class="muted" style="padding:6px 8px 10px;">Nenhum procedimento encontrado.</div>
    `;
    openSuggest();
    return;
  }

  procSuggest.innerHTML = `
    <div class="acHeader">
      <span class="chip">${view.length} sugestão(ões)</span>
      <span class="muted">Enter seleciona • Esc fecha</span>
    </div>
    ${view.map((p, i)=>`
      <div class="acItem ${i===0 ? "active":""}" role="option" aria-selected="${i===0}" data-idx="${i}">
        <div class="acMain">
          <b>${highlight(p, q)}</b>
          <small>Clique para selecionar</small>
        </div>
        <div class="acTag">Selecionar</div>
      </div>
    `).join("")}
  `;

  procSuggest.querySelectorAll(".acItem").forEach(el=>{
    el.addEventListener("mousedown", (ev)=>{
      // mousedown para não perder foco antes do click
      ev.preventDefault();
      const idx = Number(el.getAttribute("data-idx"));
      chooseSuggestion(idx);
    });
    el.addEventListener("mousemove", ()=>{
      const idx = Number(el.getAttribute("data-idx"));
      if (!Number.isNaN(idx)) setActive(idx);
    });
  });

  openSuggest();
}

function chooseSuggestion(idx){
  if (idx < 0 || idx >= SUGG_VIEW.length) return;
  const chosen = SUGG_VIEW[idx];

  procInput.value = chosen;
  state.procedimento = chosen;

  // ao escolher procedimento, limpa unidade (pra listar "onde faz")
  state.unidade = "";
  document.getElementById("unitSel").value = "";

  render();
  closeSuggest();
}

/* eventos do autocomplete */
procInput.addEventListener("input", () => {
  if (state.mode !== "procedimentos") return;
  renderSuggest(procInput.value);
});

procInput.addEventListener("focus", () => {
  if (state.mode !== "procedimentos") return;
  if (procInput.value.trim()) renderSuggest(procInput.value);
});

procInput.addEventListener("keydown", (e) => {
  if (!procSuggest.classList.contains("show")) return;

  if (e.key === "Escape"){
    e.preventDefault();
    closeSuggest();
    return;
  }
  if (e.key === "ArrowDown"){
    e.preventDefault();
    if (!SUGG_VIEW.length) return;
    setActive(Math.min(SUGG_VIEW.length - 1, SUGG_ACTIVE + 1));
    return;
  }
  if (e.key === "ArrowUp"){
    e.preventDefault();
    if (!SUGG_VIEW.length) return;
    setActive(Math.max(0, SUGG_ACTIVE - 1));
    return;
  }
  if (e.key === "Enter"){
    e.preventDefault();
    if (SUGG_ACTIVE >= 0) chooseSuggestion(SUGG_ACTIVE);
    return;
  }
});

document.addEventListener("mousedown", (e) => {
  const wrap = procInput.closest(".acWrap");
  if (!wrap) return;
  if (!wrap.contains(e.target)) closeSuggest();
});

/* =========================
   RENDER (resultado “relatório”)
   ========================= */
function render(){
  document.getElementById("chip").textContent = chipText();

  const meta = document.getElementById("meta");
  const boxMeta = document.getElementById("boxMeta");
  const boxTitle = document.getElementById("boxTitle");
  const results = document.getElementById("results");
  results.innerHTML = "";

  if (state.mode === "servicos"){
    const items = getItemsServicosFiltered();

    meta.textContent = state.municipio
      ? ("Municipio selecionado: " + state.municipio + ".")
      : "Selecione um municipio para visualizar os servicos e prestadores.";

    boxTitle.textContent = state.municipio ? ("Rede ofertada - " + state.municipio) : "Resultado (Servicos)";
    boxMeta.textContent = (state.tipoProc ? ("Tipo de Procedimento: " + state.tipoProc + ". ") : "") + ("Itens no filtro: " + items.length + ".");

    if (!items.length){
      results.innerHTML = `<div class="muted">Sem resultado para os filtros atuais.</div>`;
      renderDetailEmpty();
      refreshMapStyles();
      return;
    }

    const wrap = document.createElement("div");
    wrap.className = "tableWrap";

    wrap.innerHTML = `
      <table>
        <thead>
          <tr>
            <th>Serviço</th>
            <th>Prestador</th>
            <th style="width:160px;">Município</th>
            <th style="width:220px;">Ações</th>
          </tr>
        </thead>
        <tbody>
          ${items
            .slice()
            .sort((a,b)=> (a.servico.localeCompare(b.servico,"pt-BR")) || (a.prestador.localeCompare(b.prestador,"pt-BR")))
            .map(it => {
              const hasRol = providers.some(p => norm(p.nome) === norm(it.prestador));
              const carta = (it.link && String(it.link).trim() && it.link !== "URL_DA_CARTA_DE_SERVICOS") ? it.link : "";
              const cartaTxt = it.link === "URL_DA_CARTA_DE_SERVICOS" ? "pendente" : (carta ? "ok" : "sem");

              return `
                <tr>
                  <td><b>${escapeHtml(it.servico)}</b></td>
                  <td>${escapeHtml(it.prestador)}</td>
                  <td><span class="badge"><strong>${escapeHtml(it.municipio)}</strong></span></td>
                  <td>
                    <button class="linkBtn" type="button" ${hasRol ? "" : "disabled"} data-proc="${escapeHtml(it.prestador)}">
                      Procedimentos
                    </button>
                    ${carta ? `<a class="linkBtn" href="${carta}" target="_blank" rel="noopener">Carta</a>`
                      : `<span class="badge"><strong>Carta</strong> • ${cartaTxt}</span>`
                    }
                  </td>
                </tr>
              `;
            }).join("")}
        </tbody>
      </table>
    `;

    results.appendChild(wrap);

    results.querySelectorAll("button[data-proc]").forEach(btn => {
      btn.addEventListener("click", () => {
        const prest = btn.getAttribute("data-proc");
        buildProcPanelForPrestador(prest);
        document.getElementById("procPanel").scrollIntoView({ behavior:"smooth", block:"start" });
      });
    });

    renderDetailEmpty();
    refreshMapStyles();
    return;
  }

  // ===== PROCEDIMENTOS =====
  const list = getProvidersFiltered();
  meta.textContent = "Use unidade/perfil/procedimento para listar onde e realizado.";
  boxTitle.textContent = "Resultado (Procedimentos)";
  boxMeta.textContent =
    (state.perfil ? ("Perfil: " + state.perfil + ". ") : "") +
    (state.unidade ? ("Unidade: " + state.unidade + ". ") : "") +
    (state.procedimento ? ("Procedimento: " + state.procedimento + ". ") : "") +
    ("Unidades no filtro: " + list.length + ".");

  if (!list.length){
    results.innerHTML = `<div class="muted">Sem resultado para os filtros atuais.</div>`;
    renderDetailEmpty();
    refreshMapStyles();
    return;
  }

  const wrap = document.createElement("div");
  wrap.className = "tableWrap";
  wrap.innerHTML = `
    <table>
      <thead>
        <tr>
          <th>Unidade</th>
          <th>Município</th>
          <th>Tipo</th>
          <th style="width:160px;">Perfil</th>
          <th style="width:140px;">Ação</th>
        </tr>
      </thead>
      <tbody>
        ${list
          .slice()
          .sort((a,b)=> (a.municipio.localeCompare(b.municipio,"pt-BR")) || (a.nome.localeCompare(b.nome,"pt-BR")))
          .map(p => `
            <tr>
              <td><b>${escapeHtml(p.nome)}</b></td>
              <td><span class="badge"><strong>${escapeHtml(p.municipio)}</strong></span></td>
              <td>${escapeHtml(p.tipo || "-")}</td>
              <td><span class="badge"><strong>${escapeHtml(p.perfil || "-")}</strong></span></td>
              <td><a href="#" class="linkBtn" data-rol="${escapeHtml(p.id)}">Ver rol</a></td>
            </tr>
          `).join("")}
      </tbody>
    </table>
  `;
  results.appendChild(wrap);

  results.querySelectorAll("a[data-rol]").forEach(a => {
    a.addEventListener("click", (e) => {
      e.preventDefault();
      const id = a.getAttribute("data-rol");
      const p = providers.find(x => x.id === id);
      if (!p) return;
      state.unidade = p.nome;
      document.getElementById("unitSel").value = p.nome;
      renderProceduresDetail(p);
      refreshMapStyles();
      fitToMunicipio(p.municipio);
      pinMunicipioLabel(p.municipio);
    });
  });

  const selected = state.unidade ? providers.find(x => norm(x.nome) === norm(state.unidade)) : null;
  if (selected) renderProceduresDetail(selected); else renderDetailEmpty();

  refreshMapStyles();
}

/* =========================
   MAPA
   ========================= */
const map = L.map("map");
L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
  maxZoom: 18,
  attribution: "&copy; OpenStreetMap"
}).addTo(map);
map.setView([-10.9, -63.1], 6);

const status = document.getElementById("status");
const GEOJSON_URL = "https://raw.githubusercontent.com/tbrugz/geodata-br/master/geojson/geojs-11-mun.json";
let geoLayer = null;

let MAP_HOME_BOUNDS = null;
const MAP_HOME_OPTIONS = { padding:[18,18] };

function saveMapHomeBounds(){
  if (MAP_HOME_BOUNDS) return;
  if (!geoLayer) return;
  try{ MAP_HOME_BOUNDS = geoLayer.getBounds(); }catch(e){}
}
function resetMapToHome(){
  if (!MAP_HOME_BOUNDS) saveMapHomeBounds();
  if (!MAP_HOME_BOUNDS) return;
  try{ map.fitBounds(MAP_HOME_BOUNDS, MAP_HOME_OPTIONS); }catch(e){}
}

/* =========================
   REGIOES E CORES
   ========================= */
const REGIONS = {
  "CONE SUL": ["CABIXI","CEREJEIRAS","COLORADO DO OESTE","CORUMBIARA","CHUPINGUAIA","PIMENTEIRAS DO OESTE","VILHENA"],
  "CAFE": ["CACOAL","ESPIGAO D'OESTE","MINISTRO ANDREAZZA","PIMENTA BUENO","PRIMAVERA DE RONDONIA","SAO FELIPE D'OESTE"],
  "ZONA DA MATA": ["ALTA FLORESTA D'OESTE","ALTO ALEGRE DOS PARECIS","CASTANHEIRAS","NOVO HORIZONTE DO OESTE","NOVA BRASILANDIA D'OESTE","PARECIS","ROLIM DE MOURA","SANTA LUZIA D'OESTE"],
  "CENTRAL": ["ALVORADA D'OESTE","GOVERNADOR JORGE TEIXEIRA","JI-PARANÁ","JARU","MIRANTE DA SERRA","OURO PRETO DO OESTE","NOVA UNIAO","PRESIDENTE MEDICI","SAO MIGUEL DO GUAPORE","URUPA","VALE DO PARAISO","THEOBROMA","VALE DO ANARI","TEIXEIROPOLIS"],
  "VALE DO GUAPORE": ["COSTA MARQUES","SAO FRANCISCO DO GUAPORE","SERINGUEIRAS"],
  "VALE DO JAMARI": ["ARIQUEMES","ALTO PARAISO","BURITIS","CACAULANDIA","CAMPO NOVO DE RONDONIA","CUJUBIM","MONTE NEGRO","MACHADINHO D'OESTE","RIO CRESPO"],
  "MADEIRA MAMORE": ["CANDEIAS DO JAMARI","GUAJARA-MIRIM","ITAPUA DO OESTE","NOVA MAMORE","PORTO VELHO"]
};

const REGION_COLOR = {
  "CONE SUL":        { fill:"#7C3AED", stroke:"#ffffff" },
  "CAFE":            { fill:"#F59E0B", stroke:"#ffffff" },
  "ZONA DA MATA":    { fill:"#10B981", stroke:"#ffffff" },
  "CENTRAL":         { fill:"#44ceff", stroke:"#ffffff" },
  "VALE DO GUAPORE": { fill:"#ffff00", stroke:"#FCA5A5" },
  "VALE DO JAMARI":  { fill:"#ff098a", stroke:"#ffffff" },
  "MADEIRA MAMORE":  { fill:"#005a19", stroke:"#ffffff" }
};

const MUNICIPIO_TO_REGION = (() => {
  const map = new Map();
  Object.keys(REGIONS).forEach(region => REGIONS[region].forEach(m => map.set(norm(m), region)));
  return map;
})();

function getRegionByMunicipio(municipio){ return MUNICIPIO_TO_REGION.get(norm(municipio)) || ""; }
function getRegionColors(municipio){
  const region = getRegionByMunicipio(municipio);
  return REGION_COLOR[region] || { fill:"#2b6cff", stroke:"#35508b" };
}

function countForMunicipioName(geoName){
  const target = norm(geoName);
  const mun = allMunicipios().find(m => norm(m) === target);
  if (!mun) return { key:null, count:0 };

  if (state.mode === "servicos"){
    const tmp = { ...state, municipio: mun };
    return { key: mun, count: getItemsServicosFiltered(tmp).length };
  } else {
    const tmp = { ...state };
    const list = getProvidersFiltered(tmp).filter(p => norm(p.municipio) === norm(mun));
    return { key: mun, count: list.length };
  }
}

function styleFeature(feature){
  const nome = feature?.properties?.name || feature?.properties?.NM_MUN || "";
  const info = countForMunicipioName(nome);
  const colors = getRegionColors(nome);

  if (state.mode === "servicos" && state.municipio){
    const isTarget = info.key && norm(info.key) === norm(state.municipio);
    if (isTarget){
      return { color: colors.stroke, weight: 3, fillColor: colors.fill, fillOpacity: 0.60 };
    }
    return { color:"#223052", weight:1, fillColor:"#0b1220", fillOpacity:0.12 };
  }

  const hasFilter = (state.mode === "servicos")
    ? (state.municipio || state.tipoProc)
    : (state.perfil || state.procedimento || state.unidade);

  if (hasFilter){
    if (info.count > 0){
      return { color: colors.stroke, weight: 2, fillColor: colors.fill, fillOpacity: 0.60 };
    }
    return { color:"#223052", weight:1, fillColor:"#0b1220", fillOpacity:0.15 };
  }

  const reg = getRegionByMunicipio(nome);
  if (reg){
    return { color: colors.stroke, weight: 1, fillColor: colors.fill, fillOpacity: 0.60 };
  }

  if (info.key) return { color:"#35508b", weight:1, fillColor:"#2b6cff", fillOpacity:0.20 };
  return { color:"#223052", weight:1, fillColor:"#0b1220", fillOpacity:0.08 };
}

function refreshMapStyles(){ if (geoLayer) geoLayer.setStyle(styleFeature); }

function fitToMunicipio(municipio){
  if (!geoLayer) return;
  const target = norm(municipio);
  geoLayer.eachLayer(layer => {
    const nome = layer.feature?.properties?.name || layer.feature?.properties?.NM_MUN || "";
    if (norm(nome) === target){
      try{ map.fitBounds(layer.getBounds(), { padding:[18,18] }); }catch(e){}
    }
  });
}

/* município fixado (vermelho) */
const mapHoverEl = document.getElementById("mapHover");
let PINNED_MUN = "";

function setHoverDefault(){
  if (PINNED_MUN){
    mapHoverEl.className = "hoverStrong";
    mapHoverEl.textContent = PINNED_MUN;
  } else {
    mapHoverEl.className = "hoverDefault";
    mapHoverEl.textContent = "Passe o mouse no mapa para ver o município aqui.";
  }
}
function pinMunicipioLabel(munText){
  PINNED_MUN = munText ? String(munText) : "";
  setHoverDefault();
}

fetch(GEOJSON_URL)
  .then(r => r.json())
  .then(geo => {
    status.textContent = "Malha carregada";

    geoLayer = L.geoJSON(geo, {
      style: styleFeature,
      onEachFeature: (feature, layer) => {
        const nome = feature?.properties?.name || feature?.properties?.NM_MUN || "MUNICIPIO";

        layer.on("mouseover", (e) => {
          e.target.setStyle({ weight: 3 });
          mapHoverEl.className = "hoverHover";
          mapHoverEl.textContent = nome;
        });

        layer.on("mouseout", () => {
          refreshMapStyles();
          setHoverDefault();
        });

        layer.on("click", () => {
          const info = countForMunicipioName(nome);
          const finalName = info.key ? info.key : nome;
          pinMunicipioLabel(finalName);

          if (state.mode === "servicos"){
            state.municipio = finalName;
            document.getElementById("munSel").value = state.municipio;
            render();
          }
          fitToMunicipio(finalName);
        });
      }
    }).addTo(map);

    map.fitBounds(geoLayer.getBounds());
    saveMapHomeBounds();
    refreshMapStyles();
    setHoverDefault();
  })
  .catch(err => {
    console.error(err);
    status.textContent = "Falha ao carregar malha";
    alert("Nao foi possivel carregar o GeoJSON (internet/bloqueio).");
  });

/* =========================
   EVENTOS
   ========================= */
function setModeAndRefresh(){
  setModeUI();
  fillUI();
  renderDetailEmpty();
  render();
  refreshMapStyles();
}

fillUI();
renderDetailEmpty();
render();
setModeUI();

document.getElementById("modeSel").addEventListener("change", (e) => {
  state.mode = e.target.value || "servicos";
  closeSuggest();
  setModeAndRefresh();
});

document.getElementById("munSel").addEventListener("change", (e) => {
  state.municipio = e.target.value || "";
  render();
  if (state.municipio){
    fitToMunicipio(state.municipio);
    pinMunicipioLabel(state.municipio);
  } else {
    pinMunicipioLabel("");
  }
});

document.getElementById("tipoProcSel").addEventListener("change", (e) => {
  state.tipoProc = e.target.value || "";
  render();
});

document.getElementById("unitSel").addEventListener("change", (e) => {
  state.unidade = (e.target.value || "").trim();
  render();
});

document.getElementById("perfilSel").addEventListener("change", (e) => {
  state.perfil = (e.target.value || "").trim();
  if (state.unidade){
    const sel = providers.find(p => norm(p.nome) === norm(state.unidade));
    if (sel && state.perfil && norm(sel.perfil||"") !== norm(state.perfil)){
      state.unidade = "";
      document.getElementById("unitSel").value = "";
      renderDetailEmpty();
    }
  }
  render();
});

document.getElementById("btnLimpar").addEventListener("click", () => {
  state.municipio = "";
  state.tipoProc = "";
  state.procedimento = "";
  state.unidade = "";
  state.perfil = "";

  document.getElementById("munSel").value = "";
  document.getElementById("tipoProcSel").value = "";
  document.getElementById("procInput").value = "";
  document.getElementById("unitSel").value = "";
  document.getElementById("perfilSel").value = "";

  document.getElementById("procList").innerHTML = "";
  document.getElementById("procListFilter").value = "";
  document.getElementById("procCountChip").textContent = "0 itens";
  document.getElementById("procPanelTitle").textContent = "Procedimentos (Lista)";
  document.getElementById("procPanelMeta").textContent = "Clique em “Procedimentos” em uma unidade no Resultado para listar aqui.";
  renderProcPanelDetail(null);

  closeSuggest();
  pinMunicipioLabel("");

  renderDetailEmpty();
  render();
  refreshMapStyles();
  resetMapToHome();
});
</script>
</body>
</html>
