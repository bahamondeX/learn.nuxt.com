<script setup lang="ts">
// @ts-ignore
import hljs from 'highlight.js';
import 'highlight.js/styles/default.css';
// @ts-ignore
import MarkdownIt from 'markdown-it';

const props = defineProps({
  content: {
    type: String,
    default: '',
  },
});

// Create markdown-it instance
const md = MarkdownIt({
  html: true,
  linkify: true,
  typographer: true,
  highlight: (str: string, lang?: string): string => {
    // Create a container for the code and the copy button
    const codeId = `code-${Math.random().toString(36).substring(2, 9)}`;
    let highlighted = '';
    
    // Try to highlight the code if language is specified
    if (lang && hljs.getLanguage(lang)) {
      try {
        highlighted = hljs.highlight(str, {language: lang}).value;
      } catch (__) {
        highlighted = md.utils.escapeHtml(str);
      }
    } else {
      highlighted = md.utils.escapeHtml(str);
    }
    
    // Return HTML with code and copy button - using a more compact header
    return `
      <div class="code-block-wrapper">
        <div class="code-heading">
          <span class="language-marker">${lang || 'code'}</span>
          <button class="copy-button" data-target="${codeId}">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="14" height="14" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <rect x="8" y="8" width="12" height="12" rx="2" ry="2"></rect>
              <path d="M16 8V6a2 2 0 0 0-2-2H6a2 2 0 0 0-2 2v8a2 2 0 0 0 2 2h2"></path>
            </svg>
          </button>
        </div>
        <pre class="hljs"><code id="${codeId}">${highlighted}</code></pre>
      </div>
    `;
  },
});

// Add event listeners to copy buttons after the component is mounted
onMounted(() => {
  setTimeout(() => {
    const copyButtons = document.querySelectorAll('.copy-button');
    
    copyButtons.forEach(button => {
      button.addEventListener('click', () => {
        // Get the target code element
        const targetId = (button as HTMLElement).getAttribute('data-target');
        const codeElement = document.getElementById(targetId as string);
        
        if (codeElement) {
          // Get the text content
          const codeText = codeElement.textContent || '';
          
          // Copy to clipboard
          navigator.clipboard.writeText(codeText).then(() => {
            // Visual feedback for successful copy
            const originalHTML = (button as HTMLElement).innerHTML;
            (button as HTMLElement).innerHTML = `
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="" height="14" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M20 6L9 17l-5-5"></path>
              </svg>
            `;
            
            // Reset the button after a delay
            setTimeout(() => {
              (button as HTMLElement).innerHTML = originalHTML;
            }, 2000);
          }).catch(err => {
            console.error('Could not copy text: ', err);
          });
        }
      });
    });
  }, 100); // Small delay to ensure DOM is fully rendered
});
</script>

<template>

  <div class="markdown-content" v-html="md.render(props.content)"></div>
</template>

<style>
.markdown-content {
  text-align: left;
  width: 100%;
  max-width: 100%;
  padding: 0 16px;
  box-sizing: border-box;
}

.markdown-content h1,
.markdown-content h2,
.markdown-content h3,
.markdown-content h4,
.markdown-content h5,
.markdown-content h6 {
  text-align: left;
  margin-top: 1.5em;
  margin-bottom: 0.75em;
}

.markdown-content h1 {
  font-size: 2em;
}

.markdown-content p {
  text-align: left;
  margin: 1em 0;
  line-height: 1.6;
}

.markdown-content blockquote {
  border-left: 4px solid #ccc;
  margin-left: 0;
  padding-left: 16px;
  font-style: italic;
  color: #777;
}

.code-block-wrapper {
  position: relative;
  width: 100%;
  margin: 1.5em 0;
  background-color: #1e1e1e;
  border-radius: 6px;
  overflow: hidden;
}

.code-heading {
  position: relative;
  margin-top: 2px;
  display: flex;
  justify-content: space-between;
  align-items: top;
  background-color: #1e1e1e;
  padding: 2px 12px;
 
}

.language-marker {
  color: #777;
  font-family: monospace;
  font-size: 1em;
  line-height: 1;
}

.copy-button {
  background-color: transparent;
  border: none;
  padding: 2px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0.6;
  transition: opacity 0.2s;
  line-height: 1;
}

.copy-button:hover {
  opacity: 1;
}

.copy-button svg {
  color: #fff;
  scale: 1.5;
}

.hljs {
  width: 100%;
  margin: 0;
  overflow-x: auto;
  background-color: #1e1e1e !important;
  color: #d4d4d4 !important;
  border-radius: 0 !important;
}

.markdown-content pre {
  margin: 2em;
  padding: 0;
  width: 100%;
  background-color: transparent !important;
}

.markdown-content code {
  font-family: 'Fira Code', Consolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace;
  font-size: 0.85em;
}

.markdown-content strong, 
.markdown-content b {
  font-weight: 600;
}

.markdown-content em,
.markdown-content i {
  font-style: italic;
}
</style>