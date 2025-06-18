<template>
  <footer class="footer">
    <div class="footer-content">
      <!-- Logo y descripción -->
      <div class="footer-section">
        <h2 class="footer-title">FlowMedic</h2>
        <p>Monitorea en tiempo real la disponibilidad y tiempos de atención en hospitales de urgencias.</p>
        <div class="social-icons">
          <a href="#"><i class="fab fa-facebook-f"></i></a>
          <a href="#"><i class="fab fa-instagram"></i></a>
          <a href="#"><i class="fab fa-twitter"></i></a>
        </div>
      </div>

      <!-- Enlaces útiles -->
      <div class="footer-section">
        <h3>Servicios</h3>
        <ul>
          <li><a href="/">Inicio</a></li>
          <li><a href="/hospitales">Hospitales</a></li>
          <li><a href="/urgencias">Urgencias</a></li>
          <li><a href="/informacion">Información</a></li>
        </ul>
      </div>

      <!-- Servicios -->
      <div class="footer-section">
        <h3>Servicios</h3>
        <ul>
          <li><a href="#">Tiempos de espera</a></li>
        </ul>
      </div>

      <!-- Números de emergencia -->
      <div class="footer-section">
        <h3>Emergencias</h3>
        <ul>
          <li><strong>Arco Iris:</strong> (+591 2) 221 6021</li>
          <li><strong>Cotahuma:</strong> 2 2652767</li>
          <li><strong>Obrero N°1:</strong> (591-2) 2223392</li>
          <li><strong>Materno Infantil:</strong> (591-2) 2242424</li>
        </ul>
      </div>
    </div>

    <div class="footer-bottom">
      © 2025 <a href="#">FlowMedic</a>. Todos los derechos reservados.
    </div>
  </footer>
</template>

<script>
export default {
  name: 'FooterComponent'
}
</script>

<style scoped>
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css');

.footer {
  background: linear-gradient(to right, #efdbbf, #c96449);
  color: #121211;
  padding: 40px 20px 20px;
  font-family: 'Segoe UI', sans-serif;
}

.footer-content {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  max-width: 1200px;
  margin: auto;
  gap: 30px;
}

.footer-section {
  flex: 1;
  min-width: 200px;
}

.footer-title {
  font-size: 22px;
  font-weight: bold;
  margin-bottom: 10px;
}

.footer-section h3 {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 10px;
}

.footer-section ul {
  list-style: none;
  padding: 0;
}

.footer-section ul li {
  margin-bottom: 8px;
  font-size: 14px;
}

.footer-section ul li a {
  color: #121211;
  text-decoration: none;
}

.footer-section ul li a:hover {
  text-decoration: underline;
}

.social-icons {
  margin-top: 10px;
}

.social-icons a {
  margin-right: 10px;
  color: white;
  font-size: 18px;
  background: #ac3030;
  padding: 8px;
  border-radius: 50%;
  display: inline-block;
  transition: background 0.3s;
}

.social-icons a:hover {
  background: #68181f;
}

.footer-bottom {
  margin-top: 40px;
  text-align: center;
  padding-top: 10px;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
  font-size: 14px;
}

.footer-bottom a {
  color: #121211;
  text-decoration: none;
}

.footer-bottom a:hover {
  text-decoration: underline;
}
</style>
