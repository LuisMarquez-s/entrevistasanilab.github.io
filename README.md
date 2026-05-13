<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <style>
    body { 
      font-family: Arial, sans-serif; 
      background: #0f172a; 
      color: white; 
      text-align: center; 
      padding: 20px; 
    }
    .box { 
      background: #1e293b; 
      padding: 25px; 
      border-radius: 12px; 
      max-width: 720px; 
      margin: auto; 
      box-shadow: 0 4px 15px rgba(0,0,0,0.3); 
    }
    input, select, textarea { 
      padding: 12px; 
      width: 85%; 
      margin: 10px auto; 
      display: block; 
      border-radius: 6px; 
      border: none; 
      font-size: 16px; 
    }
    button { 
      padding: 12px 28px; 
      margin: 10px 5px; 
      background: #22c55e; 
      border: none; 
      color: white; 
      border-radius: 6px; 
      cursor: pointer; 
      font-size: 16px; 
      font-weight: bold; 
    }
    button.atras { background: #64748b; }
    .tiktok-player { 
      width: 100%; 
      max-width: 380px; 
      height: 620px; 
      margin: 15px auto; 
      border-radius: 12px; 
      overflow: hidden; 
      background: #000; 
    }
    .cam-box { 
      width: 260px; 
      height: 260px; 
      margin: 15px auto; 
      border-radius: 12px; 
      overflow: hidden; 
      border: 3px solid #22c55e; 
      background: black; 
    }
    .cam-box video { 
      width: 100%; 
      height: 100%; 
      object-fit: cover; 
    }
    .resumen { 
      text-align: left; 
      background: #334155; 
      padding: 18px; 
      border-radius: 10px; 
      margin: 20px auto; 
      max-width: 550px; 
      line-height: 1.6; 
    }
    .warning { color: #eab308; font-weight: bold; }
    canvas { 
      border: 2px solid #22c55e; 
      border-radius: 8px; 
      background: white; 
    }
  </style>
</head>
<body>
  <div class="box" id="app"></div>

  <script src="https://cdn.jsdelivr.net/npm/signature_pad@4.0.0/dist/signature_pad.umd.min.js"></script>
  
  <script>
    const API_URL = "https://script.google.com/macros/s/AKfycbxqB9Fk-ZS_jK93zGCOBwte0c4wKWzWLGfqpJeComTLEp1ZoRu-OmY5nGlOTmID1rjm/exec";

    let paso = 1;
    let datos = {
      nombre: "", email: "", telefono: "", tipoUsuario: "", subTipo: "",
      dias: [], horaInicio: "", horaFin: "", modalidad: "", ubicacion: "",
      diasPresencial: [], diasRemoto: [], cambios: "", firmaUrl: "",
      videoPresentacion: "", videoProductividad: "", videoRetiro: ""
    };

    let signaturePad, stream, mediaRecorder, chunks = [];

    // Función genérica para enviar datos a la API de Apps Script
    async function enviarDatosAPI(payload) {
      try {
        const respuesta = await fetch(API_URL, {
          method: 'POST',
          headers: { 'Content-Type': 'text/plain;charset=utf-8' },
          body: JSON.stringify(payload)
        });
        return await respuesta.json();
      } catch (error) {
        console.error("Error en la conexión con la API:", error);
        return { success: false, message: "Error de red al conectar con el servidor." };
      }
    }

    function render() {
      let html = "";

      switch(paso) {
        case 1:
          html = `
            <h2>👋 Hola. Bienvenido al Sistema de Entrevista de Sanilab</h2>
            <p>Ingresa tu nombre y apellido completo.Pero primero mira este pequeño video para que nos conozcas mejor:</p>
            <div class="tiktok-player">
              <iframe
              src="https://drive.google.com/file/d/18hHbql960CwoQPOu8l2vXeX6AJJXJYc7/preview"
              width="100%"
              height="100%"
               frameborder="0"
               allow="autoplay; fullscreen"
               allowfullscreen>
              </iframe>
            </div>
            <input id="nombre" placeholder="Nombre completo *" value="${datos.nombre}">
            <button onclick="guardarNombre()">Continuar</button>
          `;
          break;

        case 2:
          html = `
            <h2>🎥 Preséntate</h2>
            <p>Ahora queremos conocerte mejor 😊. Mira este video y luego graba tu video de presentación (sé claro y natural).</p>
            <div class="tiktok-player">
              <iframe src="https://www.tiktok.com/player/v1/7340360993362447622" width="100%" height="100%" frameborder="0" allowfullscreen></iframe>
            </div>
            <div class="cam-box"><video id="video" autoplay playsinline></video></div>
            <button onclick="startCam('video')">📷 Activar cámara</button>
            <button onclick="startRec(1)" id="btnGrabar">🎥 Grabar</button>
            <button onclick="stopRec()" id="btnDetener" disabled>⏹ Detener</button>
            <div id="preview"></div>
            <p><strong>Descarga el video de presentación.</strong> Solo puede visualizarse una vez.</p>
            <button onclick="descargarVideo()">⬇️ Descargar Video de Presentación</button>
            <br><br>
            <button onclick="next()">Continuar</button>
            <button onclick="prev()" class="atras">← Atrás</button>
          `;
          break;

        case 3:
          html = `
            <h2>🧾 Tus Datos</h2>
            <p>“Perfecto 👍.Completa tu correo electrónico y número de teléfono.Si no tienes correo,mira como hacer uno con el siguiente video:.</p>
            <div class="tiktok-player">
              <iframe src="https://www.tiktok.com/player/v1/7601894722256244023" width="100%" height="100%" frameborder="0" allowfullscreen></iframe>
            </div>
            <input id="email" type="email" placeholder="Correo electrónico *" value="${datos.email}">
            <input id="telefono" placeholder="Teléfono *" value="${datos.telefono}">
            <button onclick="saveData()">Continuar</button>
            <button onclick="prev()" class="atras">← Atrás</button>
          `;
          break;

        case 4:
          html = `
            <h2>🧑‍🎓 Tipo de Usuario</h2>
            <p>Elige la opción que mejor describe tu postulación con el siguiente video::</p>
            <div class="tiktok-player">
              <iframe src="https://www.tiktok.com/player/v1/7496168002094746886" width="100%" height="100%" frameborder="0" allowfullscreen></iframe>
            </div>
            <select id="tipoUsuario" onchange="actualizarTipo()">
              <option value="">Selecciona...</option>
              <option value="Estudiante" ${datos.tipoUsuario==='Estudiante'?'selected':''}>Estudiante</option>
              <option value="Egresado" ${datos.tipoUsuario==='Egresado'?'selected':''}>Egresado</option>
            </select>
            
            <div id="subtipoDiv" style="display:${datos.tipoUsuario==='Estudiante'?'block':'none'}">
              <p>Tambien elige si quieres ser:</p>
              <select id="subTipo">
                <option value="Pasante" ${datos.subTipo==='Pasante'?'selected':''}>Pasante</option>
                <option value="Practicante" ${datos.subTipo==='Practicante'?'selected':''}>Practicante</option>
                <option value="Remunerado" ${datos.subTipo==='Remunerado'?'selected':''}>Remunerado</option>
              </select>
            </div>
            
            <div id="egresadoDiv" style="display:${datos.tipoUsuario==='Egresado'?'block':'none'}; color:#eab308; margin:15px;">
              <strong>Modalidad remunerada</strong> – pago sujeto a negociación con el gerente.
            </div>
            
            <button onclick="guardarTipo()">Continuar</button>
            <button onclick="prev()" class="atras">← Atrás</button>
          `;
          break;

        case 5:
          html = `
            <h2>⏰ Disponibilidad de Horario</h2>
            <p>Selecciona tus días disponibles:</p>
            <div style="text-align:left; max-width:420px; margin:15px auto;">
              ${['Lunes','Martes','Miércoles','Jueves','Viernes','Sábado','Domingo'].map(dia => `
                <label><input type="checkbox" value="${dia}" ${datos.dias.includes(dia)?'checked':''}> ${dia}</label><br>
              `).join('')}
            </div>
            <input type="time" id="horaInicio" value="${datos.horaInicio}">
            <input type="time" id="horaFin" value="${datos.horaFin}">
            <button onclick="guardarHorario()">Continuar</button>
            <button onclick="prev()" class="atras">← Atrás</button>
          `;
          break;

        case 6:
          html = `
            <h2>🏢 Modalidad de Trabajo</h2>
            <select id="modalidad" onchange="actualizarModalidad()">
              <option value="">Selecciona...</option>
              <option value="Presencial" ${datos.modalidad==='Presencial'?'selected':''}>Presencial</option>
              <option value="Remoto" ${datos.modalidad==='Remoto'?'selected':''}>Remoto</option>
              <option value="Híbrido" ${datos.modalidad==='Híbrido'?'selected':''}>Híbrido</option>
            </select>
            
            <div id="ubicacionDiv" style="display:${datos.modalidad==='Presencial'?'block':'none'}">
              <select id="ubicacion">
                <option value="Miraflores" ${datos.ubicacion==='Miraflores'?'selected':''}>Miraflores</option>
                <option value="Campo" ${datos.ubicacion==='Campo'?'selected':''}>Campo</option>
              </select>
            </div>
            
            <div id="hibridoDiv" style="display:${datos.modalidad==='Híbrido'?'block':'none'}">
              <p>Días Presencial:</p>
              <select id="diasPresencial" multiple style="height:110px;">
                ${['Lunes','Martes','Miércoles','Jueves','Viernes'].map(d => 
                  `<option value="${d}" ${datos.diasPresencial.includes(d)?'selected':''}>${d}</option>`
                ).join('')}
              </select>
              <p>Días Remoto:</p>
              <select id="diasRemoto" multiple style="height:110px;">
                ${['Lunes','Martes','Miércoles','Jueves','Viernes'].map(d => 
                  `<option value="${d}" ${datos.diasRemoto.includes(d)?'selected':''}>${d}</option>`
                ).join('')}
              </select>
            </div>
            
            <button onclick="guardarModalidad()">Continuar</button>
            <button onclick="prev()" class="atras">← Atrás</button>
          `;
          break;

      case 7:
          html = `
            <h2>📱 Video Informativo</h2>
            <p>Mira el video correspondiente a tu perfil:</p>
            <div class="tiktok-player">
              <iframe src="https://www.tiktok.com/player/v1/7537859903172414728" width="100%" height="100%" frameborder="0" allowfullscreen></iframe>
            </div>
            <p class="warning">Este video solo puede reproducirse una vez.</p>
            <button onclick="marcarVideoVisto()">✅ He visto el video</button>
            <button onclick="prev()" class="atras">← Atrás</button>
          `;
          break;    

        case 8:
          html = `
            <h2>✍️ Solicitud de Cambios</h2>
            <p>Si requiere algún cambio ahora o en el futuro, indíquelo:</p>
            <textarea id="cambios" rows="5" placeholder="Ejemplo: Prefiero entrar más temprano los miércoles...">${datos.cambios || ''}</textarea>
            <button onclick="guardarCambios()">Continuar</button>
            <button onclick="prev()" class="atras">← Atrás</button>
          `;
          break;

        case 9:
          html = `
            <h2>📈 Productividad</h2>
            <p>Mira este consejo rápido de productividad antes de continuar:.</p>
            <div class="tiktok-player">
              <iframe
              src="https://drive.google.com/file/d/1kt7xypCyu-y89HYCykK1hK8qA7uvoPa8/preview"
              width="100%"
              height="100%"
               frameborder="0"
                allow="autoplay; fullscreen"
                   allowfullscreen>
               </iframe>
            </div>
            <div class="cam-box"><video id="video2" autoplay playsinline></video></div>
            <button onclick="startCam('video2')">📷 Activar cámara</button>
            <button onclick="startRec(2)" id="btnGrabar">🎥 Grabar</button>
            <button onclick="stopRec()" id="btnDetener" disabled>⏹ Detener</button>
            <div id="preview"></div>
            <button onclick="next()">Continuar</button>
            <button onclick="prev()" class="atras">← Atrás</button>
          `;
          break;

        case 10:
          html = `
            <h2>🚪 Retiro de tu último empleo</h2>
            <p>Mira cómo responder positivamente esta pregunta común:.</p>
            <div class="tiktok-player">
              <iframe src="https://www.tiktok.com/player/v1/7244346048775228678" width="100%" height="100%" frameborder="0" allowfullscreen></iframe>
            </div>
            <div class="cam-box"><video id="video3" autoplay playsinline></video></div>
            <button onclick="startCam('video3')">📷 Activar cámara</button>
            <button onclick="startRec(3)" id="btnGrabar">🎥 Grabar</button>
            <button onclick="stopRec()" id="btnDetener" disabled>⏹ Detener</button>
            <div id="preview"></div>
            <button onclick="next()">Continuar</button>
            <button onclick="prev()" class="atras">← Atrás</button>
          `;
          break;

        case 11:
          html = `
            <h2>✍ Firma Digital</h2>
            <p>Firma claramente en el recuadro.</p>
            <div class="tiktok-player">
              <iframe src="https://www.tiktok.com/player/v1/7245295841550978350" width="100%" height="100%" frameborder="0" allowfullscreen></iframe>
            </div>
            <canvas id="canvasFirma" width="420" height="180"></canvas>
            <br>
            <button onclick="limpiarFirma()">🗑️ Limpiar</button>
            <button onclick="guardarFirma(event)">Guardar Firma y Continuar</button>
            <button onclick="prev()" class="atras">← Atrás</button>
          `;
          break;

        case 12:
          html = `
            <h2>✅ Resumen Final de tu Registro</h2>
            <div class="resumen">
              <p><strong>Nombre:</strong> ${datos.nombre}</p>
              <p><strong>Tipo:</strong> ${datos.tipoUsuario} ${datos.subTipo ? `(${datos.subTipo})` : ''}</p>
              <p><strong>Días:</strong> ${datos.dias.join(', ') || 'No especificado'}</p>
              <p><strong>Horario:</strong> ${datos.horaInicio || '--'} - ${datos.horaFin || '--'}</p>
              <p><strong>Modalidad:</strong> ${datos.modalidad} ${datos.ubicacion ? `(${datos.ubicacion})` : ''}</p>
              <p><strong>Cambios solicitados:</strong> ${datos.cambios || 'Ninguno'}</p>
            </div>
            <p><strong>Instrucciones finales:</strong></p>
            <ul style="padding-left: 20px; line-height: 1.6; text-align:left; max-width:500px; margin:15px auto;">
              <li>Envía este registro al grupo general de WhatsApp: 
                <a href="https://chat.whatsapp.com/FpHkGyZQ6R8B7aPsiSNqKw" target="_blank" style="color:#22c55e;">Abrir grupo</a>
              </li>
              <li>Adjunta el video de presentación descargado.</li>
              <li>Examina este documento importante:
                <a href="https://docs.google.com/document/u/0/d/164G1ekRdqKTlzmypqCpSiYkYmMWqLza4d_1r7SILivE/mobilebasic" target="_blank" style="color:#22c55e;">Ver Documento</a>
              </li>
            </ul>
            <button onclick="finalizarRegistro()" style="background:#22c55e; padding:14px 32px; font-size:18px;">Finalizar Entrevista</button>
            <br><br>
            <button onclick="prev()" class="atras">← Atrás</button>
          `;
          break;

        default:
          html = `<h2>🎉 ¡Registro Completado!</h2><p>Gracias por registrarte en Sanilab.</p>`;
      }

      document.getElementById("app").innerHTML = html;

      if (paso === 11) {
        setTimeout(() => {
          const canvas = document.getElementById("canvasFirma");
          if (canvas) signaturePad = new SignaturePad(canvas);
        }, 100);
      }
    }

    function apagarCamara() {
      if (stream) {
        stream.getTracks().forEach(track => track.stop());
        stream = null;
      }
    }

    async function startCam(id) {
      try {
        stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" }, audio: true });
        document.getElementById(id).srcObject = stream;
      } catch(e) {
        alert("Por favor activa los permisos de cámara y micrófono.");
      }
    }

    function startRec(tipo) {
      if (!stream) return alert("Primero activa la cámara");
      
      // Control de botones
      const btnGrabar = document.getElementById("btnGrabar");
      const btnDetener = document.getElementById("btnDetener");
      if(btnGrabar) btnGrabar.disabled = true;
      if(btnDetener) btnDetener.disabled = false;

      chunks = [];
      mediaRecorder = new MediaRecorder(stream);
      mediaRecorder.ondataavailable = e => chunks.push(e.data);

      mediaRecorder.onstop = async () => {
        const blob = new Blob(chunks, { type: 'video/webm' });
        const url = URL.createObjectURL(blob);
        document.getElementById("preview").innerHTML = `<video src="${url}" controls width="100%"></video><p style="color:#eab308;">⏳ Subiendo video a Google Drive...</p>`;

        let reader = new FileReader();
        reader.onloadend = async function () {
          const base64data = reader.result.split(",");
          const payload = {
            accion: "guardarVideo",
            base64: base64data,
            tipo: tipo,
            nombre: datos.nombre
          };

          const respuesta = await enviarDatosAPI(payload);

          if(respuesta.success){
            document.getElementById("preview").innerHTML = `<video src="${url}" controls width="100%"></video><p style="color:#22c55e;">✅ Video guardado correctamente</p>`;
            if(tipo === 1) datos.videoPresentacion = respuesta.url;
            if(tipo === 2) datos.videoProductividad = respuesta.url;
            if(tipo === 3) datos.videoRetiro = respuesta.url;
          } else {
            alert("Error al subir el video. Intenta de nuevo.");
            document.getElementById("preview").innerHTML += `<p style="color:red;">❌ Falló la subida</p>`;
          }
        };
        reader.readAsDataURL(blob);
      };

      mediaRecorder.start();
    }

    function stopRec() {
      if (mediaRecorder) mediaRecorder.stop();
      const btnGrabar = document.getElementById("btnGrabar");
      const btnDetener = document.getElementById("btnDetener");
      if(btnGrabar) btnGrabar.disabled = false;
      if(btnDetener) btnDetener.disabled = true;
      apagarCamara();
    }

    function descargarVideo() {
      if (chunks.length === 0) return alert("Primero debes grabar un video.");
      const blob = new Blob(chunks, { type: 'video/webm' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.style.display = 'none';
      a.href = url;
      a.download = `Presentacion_${datos.nombre || 'Sanilab'}.webm`;
      document.body.appendChild(a);
      a.click();
      setTimeout(() => { document.body.removeChild(a); window.URL.revokeObjectURL(url); }, 100);
    }

    function guardarNombre() { datos.nombre = document.getElementById("nombre").value.trim(); if(!datos.nombre) return alert("Por favor ingresa tu nombre completo"); next(); }
    function saveData() { datos.email = document.getElementById("email").value.trim(); datos.telefono = document.getElementById("telefono").value.trim(); if (!datos.email || !datos.telefono) return alert("Completa el correo y teléfono"); next(); }
    function actualizarTipo() { datos.tipoUsuario = document.getElementById("tipoUsuario").value; render(); }
    function guardarTipo() { datos.tipoUsuario = document.getElementById("tipoUsuario").value; if (datos.tipoUsuario === "Estudiante") { datos.subTipo = document.getElementById("subTipo").value; } else { datos.subTipo = ""; } if (!datos.tipoUsuario) return alert("Selecciona un tipo de usuario"); next(); }
    function guardarHorario() { datos.dias = Array.from(document.querySelectorAll('input[type="checkbox"]:checked')).map(cb => cb.value); datos.horaInicio = document.getElementById("horaInicio").value; datos.horaFin = document.getElementById("horaFin").value; if (!datos.horaInicio || !datos.horaFin) return alert("Ingresa hora de inicio y fin"); next(); }
    function actualizarModalidad() { datos.modalidad = document.getElementById("modalidad").value; render(); }
    function guardarModalidad() { datos.modalidad = document.getElementById("modalidad").value; if (datos.modalidad === "Presencial") { datos.ubicacion = document.getElementById("ubicacion").value; } else if (datos.modalidad === "Híbrido") { datos.diasPresencial = Array.from(document.getElementById("diasPresencial").selectedOptions).map(o => o.value); datos.diasRemoto = Array.from(document.getElementById("diasRemoto").selectedOptions).map(o => o.value); } next(); }
    function guardarCambios() { datos.cambios = document.getElementById("cambios").value.trim(); next(); }
    function limpiarFirma() { if (signaturePad) signaturePad.clear(); }

    async function guardarFirma(e) {
      if (!signaturePad || signaturePad.isEmpty()) return alert("Por favor realiza tu firma digital");
      const boton = e.target;
      boton.innerText = "⏳ Guardando Firma...";
      boton.disabled = true;

      const payload = {
        accion: "subirFirma",
        base64: signaturePad.toDataURL(),
        nombre: datos.nombre
      };

      const respuesta = await enviarDatosAPI(payload);

      if (respuesta.success) {
        datos.firmaUrl = respuesta.url;
        next();
      } else {
        alert("Error al guardar la firma.");
        boton.innerText = "Guardar Firma y Continuar";
        boton.disabled = false;
      }
    }

    async function finalizarRegistro() {
      const app = document.getElementById("app");
      app.innerHTML = `<h2>⏳ Guardando tu información...</h2><p>Espera unos segundos. No cierres esta ventana.</p>`;

      const payload = {
        accion: "guardarRegistro",
        datos: datos
      };

      const respuesta = await enviarDatosAPI(payload);

      if (respuesta.success) {
        app.innerHTML = `
          <h2>🎉 ¡Registro Completado!</h2>
          <p>${respuesta.message}</p><br>
          <p>Gracias por completar tu proceso en Sanilab.</p>
          <p>Envía ahora tu video de presentación descargado al grupo de WhatsApp.</p>
          <a href="https://chat.whatsapp.com/FpHkGyZQ6R8B7aPsiSNqKw" target="_blank">
            <button style="background:#22c55e;">Abrir Grupo WhatsApp</button>
          </a>
        `;
      } else {
        app.innerHTML = `<h2>⚠ Error</h2><p>${respuesta.message}</p><button onclick="render()">Volver a intentar</button>`;
      }
    }

    function next() { apagarCamara(); paso++; render(); }
    function prev() { apagarCamara(); if (paso > 1) { paso--; render(); } }

    render();
  </script>
</body>
</html>
