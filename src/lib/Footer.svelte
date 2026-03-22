<footer>
  <div class="footer-content">
    <p>&copy; 2024 SwiftResize. All rights reserved.</p>
    <div class="footer-links">
      <a href="#privacy">Privacy</a>
      <span class="divider">•</span>
      <a href="#terms">Terms</a>
      <span class="divider">•</span>
      <a href="#contact">Contact</a>
    </div>
  </div>
</footer>

<style>
  footer {
    background: var(--accent-bg);
    border-top: 1px solid var(--accent-border);
    padding: 24px 20px;
    text-align: center;
    margin-top: 48px;
  }

  .footer-content {
    width: 100%;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  p {
    margin: 0 0 12px 0;
    color: var(--text);
    font-size: 13px;
  }

  .footer-links {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 8px;
  }

  a {
    color: var(--accent);
    text-decoration: none;
    font-size: 12px;
    transition: opacity 0.2s ease;

    &:hover {
      opacity: 0.8;
    }
  }

  .divider {
    color: var(--text);
  }
</style>
