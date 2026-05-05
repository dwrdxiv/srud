<template>
  <footer class="footer" style="font-family: Arial, Helvetica, sans-serif;">
    <div class="footer-content">
      
      <div class="footer-section">
        <h3>About this project:</h3>
        <p>
          This inventory system is a crud project developed as a portfolio piece
          to show my skills in Fullstack development with Vue 3 and Firebase.
          First project of this kind, so I hope you like it and if you have any feedback or suggestions, please let me know!
        </p>
        <p class="disclaimer">
          * This project is for educational purposes only and is not intended for commercial use. All data is stored in a Firebase database, and there is no authentication implemented, so please do not enter any sensitive information.
        </p>
      </div>

      
      <div class="footer-section">
        <h3>Contact</h3>
        <ul class="contact-links">
          <li>
            <a href="https://www.linkedin.com/in/dwrdxiv/" target="_blank">
              <i class="fab fa-linkedin"></i> LinkedIn
            </a>
          </li>
          <li>
            <a href="https://github.com/dwrdxiv" target="_blank">
              <i class="fab fa-github"></i> GitHub
            </a>
          </li>
        </ul>
      </div>
    </div>
    
    <div class="footer-bottom">
      <p>&copy; 2026 - by Edward Quintero (dwrdxiv)</p>
    </div>
  </footer>
</template>

<style scoped>
.footer {
  background-image: linear-gradient(to bottom, #010d18 60%, #021c35);
  color: white;
  padding: 40px 0 20px 0;
  margin-top: 50px;
}

.footer-content {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
  max-width: 1200px;
  margin: 0 auto;
}

.footer-section {
  flex: 1;
  padding: 20px;
  min-width: 250px;
}

.disclaimer {
  font-size: 0.8rem;
  color: #bdc3c7;
  margin-top: 10px;
}

.contact-links {
  list-style: none;
  padding: 0;
}

.contact-links li {
  margin-bottom: 10px;
}

.contact-links a {
  color: #21a6ff;
  text-decoration: none;
  transition: all 0.3s ease;
}
.contact-links a:hover {
  text-decoration: underline;
  font-size: 1.1rem;
}

.footer-bottom {
  text-align: center;
  border-top: 1px solid #3e4f5f;
  margin-top: 20px;
  padding-top: 20px;
}
</style>