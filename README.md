document.addEventListener('DOMContentLoaded', () => {
    // State
    let currentLevel = '0';
    let currentModule = 'dashboard';

    // DOM Elements
    const levelSelect = document.getElementById('level-select');
    const navLinks = document.querySelectorAll('.nav-links li');
    const moduleTitle = document.getElementById('module-title');
    const contentArea = document.getElementById('content-area');
    const themeToggle = document.getElementById('theme-toggle');

    // Speech Synthesis
    const synth = window.speechSynthesis;

    function speak(text) {
        if (synth.speaking) {
            console.error('speechSynthesis.speaking');
            return;
        }
        if (text !== '') {
            const utterThis = new SpeechSynthesisUtterance(text);
            utterThis.lang = 'en-US'; // English pronunciation only
            utterThis.rate = 0.9; // Slightly slower for clear English learning
            synth.speak(utterThis);
        }
    }

    // Event Listeners
    levelSelect.addEventListener('change', (e) => {
        currentLevel = e.target.value;
        renderModule();
    });

    navLinks.forEach(link => {
        link.addEventListener('click', (e) => {
            navLinks.forEach(l => l.classList.remove('active'));
            e.currentTarget.classList.add('active');
            currentModule = e.currentTarget.dataset.module;
            moduleTitle.textContent = e.currentTarget.textContent.trim();
            renderModule();
        });
    });

    themeToggle.addEventListener('click', () => {
        const body = document.body;
        if (body.dataset.theme === 'light') {
            body.dataset.theme = 'dark';
            themeToggle.innerHTML = '<i class="fa-solid fa-sun"></i>';
        } else {
            body.dataset.theme = 'light';
            themeToggle.innerHTML = '<i class="fa-solid fa-moon"></i>';
        }
    });

    // Render Logic
    function renderModule() {
        contentArea.innerHTML = ''; // Clear current content
        const levelData = appData.levels[currentLevel];

        if (!levelData) {
            contentArea.innerHTML = `<p>Data for Level ${currentLevel} coming soon.</p>`;
            return;
        }

        switch(currentModule) {
            case 'dashboard':
                renderDashboard(levelData);
                break;
            case 'vocabulary':
                renderVocabulary(levelData.vocabulary);
                break;
            case 'grammar':
                renderGrammar(levelData.grammar);
                break;
            case 'conversation':
                renderConversation(levelData.conversation);
                break;
            case 'listening':
                renderListening(levelData.listening);
                break;
            case 'phonetics':
                renderPhonetics(levelData.phonetics);
                break;
        }
    }

    function renderDashboard(data) {
        contentArea.innerHTML = `
            <div class="welcome-screen">
                <h2>Welcome to Level ${currentLevel}</h2>
                <p style="margin-top: 1rem; color: var(--text-muted);">${data.description || 'Continue your learning journey.'}</p>
                <div class="topic-grid" style="margin-top: 2rem;">
                    <div class="topic-card" onclick="document.querySelector('[data-module=vocabulary]').click()">
                        <i class="fa-solid fa-book-open"></i>
                        <h3>Vocabulary</h3>
                    </div>
                    <div class="topic-card" onclick="document.querySelector('[data-module=grammar]').click()">
                        <i class="fa-solid fa-scale-balanced"></i>
                        <h3>Grammar</h3>
                    </div>
                    <div class="topic-card" onclick="document.querySelector('[data-module=conversation]').click()">
                        <i class="fa-solid fa-comments"></i>
                        <h3>Conversation</h3>
                    </div>
                    <div class="topic-card" onclick="document.querySelector('[data-module=listening]').click()">
                        <i class="fa-solid fa-headphones"></i>
                        <h3>Listening</h3>
                    </div>
                    <div class="topic-card" onclick="document.querySelector('[data-module=phonetics]').click()">
                        <i class="fa-solid fa-microphone-lines"></i>
                        <h3>Phonetics</h3>
                    </div>
                </div>
            </div>
        `;
    }

    function renderVocabulary(vocabData) {
        if (!vocabData || vocabData.length === 0) {
            contentArea.innerHTML = '<p>No vocabulary found for this level.</p>';
            return;
        }

        let html = '<div class="topic-grid">';
        vocabData.forEach(topic => {
            html += `
                <div class="topic-card" data-topic-id="${topic.id}">
                    <i class="fa-solid fa-layer-group"></i>
                    <h3>${topic.topic}</h3>
                    <p class="arabic-text">${topic.words.length} كلمات</p>
                </div>
            `;
        });
        html += '</div>';
        
        contentArea.innerHTML = html;

        // Attach event listeners
        contentArea.querySelectorAll('.topic-card').forEach(card => {
            card.addEventListener('click', (e) => {
                const topicId = e.currentTarget.dataset.topicId;
                const selectedTopic = vocabData.find(t => t.id === topicId);
                renderVocabularyTopic(selectedTopic);
            });
        });
    }

    function renderVocabularyTopic(topic) {
        contentArea.innerHTML = `
            <div>
                <button class="btn" style="margin-bottom: 1rem;" id="back-vocab">
                    <i class="fa-solid fa-arrow-left"></i> Back to Topics
                </button>
            </div>
            <div id="flashcard-container"></div>
        `;
        document.getElementById('back-vocab').addEventListener('click', () => renderModule());

        const flashcardContainer = document.getElementById('flashcard-container');
        let currentWordIndex = 0;

        function updateCard() {
            const word = topic.words[currentWordIndex];
            flashcardContainer.innerHTML = `
                <div class="flashcard">
                    <div style="width: 100%; height: 200px; background: rgba(0,0,0,0.2); border-radius: 12px; display:flex; justify-content:center; align-items:center; color: var(--text-muted); border: 1px dashed var(--glass-border);">
                        <i class="fa-regular fa-image" style="font-size: 4rem;"></i>
                        <!-- Placeholder for image: <img src="\${word.img}" alt="\${word.en}" class="flashcard-image"> -->
                    </div>
                    <div class="flashcard-en">
                        ${word.en} 
                        <button class="btn sound-btn" onclick="window.speakTarget('${word.en.replace(/'/g, "\\'")}')">
                            <i class="fa-solid fa-volume-high"></i>
                        </button>
                    </div>
                    <div class="flashcard-ar arabic-text">${word.ar}</div>
                    <div style="display: flex; gap: 1rem; margin-top: 1rem;">
                        <button class="btn" id="prev-btn" ${currentWordIndex === 0 ? 'disabled' : ''}>Previous</button>
                        <button class="btn" id="next-btn" ${currentWordIndex === topic.words.length - 1 ? 'disabled' : ''}>Next</button>
                    </div>
                </div>
            `;
            
            const prevBtn = document.getElementById('prev-btn');
            const nextBtn = document.getElementById('next-btn');

            if(prevBtn) prevBtn.addEventListener('click', () => { currentWordIndex--; updateCard(); });
            if(nextBtn) nextBtn.addEventListener('click', () => { currentWordIndex++; updateCard(); });
        }

        // Expose speak function globally for inline handlers if needed
        window.speakTarget = speak;
        updateCard();
    }

    function renderGrammar(grammarData) {
        if (!grammarData) {
            contentArea.innerHTML = '<p>No grammar found.</p>';
            return;
        }

        let html = '<div style="display: flex; flex-direction: column; gap: 2rem;">';
        
        // Verbs
        html += '<section><h2>Verbs & Conjugations</h2><div class="topic-grid">';
        grammarData.verbs.forEach(v => {
            html += `
                <div class="glass-panel" style="padding: 1.5rem; border-radius: 12px;">
                    <h3>${v.en} <span class="arabic-text" style="font-size: 1rem; color: var(--text-muted);">(${v.ar})</span></h3>
                    <p style="margin: 1rem 0; color: var(--accent-color);">${v.rules}</p>
                    ${v.examples ? `<ul style="margin-left: 1.5rem; color: var(--text-muted);">${v.examples.map(ex => `<li>${ex}</li>`).join('')}</ul>` : ''}
                </div>
            `;
        });
        html += '</div></section>';

        // Irregular Verbs
        if(grammarData.irregular_verbs && grammarData.irregular_verbs.length > 0) {
            html += '<section><h2>Irregular Verbs</h2><div style="overflow-x:auto;"><table style="width:100%; text-align:left; border-collapse: collapse;">';
            html += '<tr style="border-bottom: 1px solid var(--glass-border);"><th>Base Form</th><th>Past Simple</th><th>Past Participle</th><th class="arabic-text" style="text-align:right;">Meaning</th></tr>';
            grammarData.irregular_verbs.forEach(v => {
                html += `<tr style="border-bottom: 1px solid var(--glass-border);">
                    <td style="padding: 0.5rem 0;">${v.base}</td>
                    <td>${v.past}</td>
                    <td>${v.participle}</td>
                    <td class="arabic-text" style="text-align:right; color: var(--text-muted);">${v.ar}</td>
                </tr>`;
            });
            html += '</table></div></section>';
        }

        // Nouns
        html += '<section><h2>Noun Patterns & Order</h2><div class="topic-grid">';
        grammarData.nouns.forEach(n => {
            html += `
                <div class="glass-panel" style="padding: 1.5rem; border-radius: 12px;">
                    <h3>${n.en} <span class="arabic-text" style="font-size: 1rem; color: var(--text-muted);">(${n.ar})</span></h3>
                    <p style="margin: 1rem 0; color: var(--accent-color);">${n.rules}</p>
                    ${n.examples ? `<ul style="margin-left: 1.5rem; color: var(--text-muted);">${n.examples.map(ex => `<li>${ex}</li>`).join('')}</ul>` : ''}
                </div>
            `;
        });
        html += '</div></section>';

        html += '</div>';
        contentArea.innerHTML = html;
    }

    function renderConversation(conversationData) {
        if (!conversationData || conversationData.length === 0) {
            contentArea.innerHTML = '<p>No conversations found for this level.</p>';
            return;
        }

        let html = '<div class="topic-grid">';
        conversationData.forEach(conv => {
            html += `
                <div class="topic-card" data-conv-id="${conv.id}">
                    <i class="fa-solid fa-comments"></i>
                    <h3>${conv.title}</h3>
                </div>
            `;
        });
        html += '</div>';
        
        contentArea.innerHTML = html;

        contentArea.querySelectorAll('.topic-card').forEach(card => {
            card.addEventListener('click', (e) => {
                const convId = e.currentTarget.dataset.convId;
                const selectedConv = conversationData.find(c => c.id === convId);
                renderConversationDetail(selectedConv);
            });
        });
    }

    function renderConversationDetail(conv) {
        contentArea.innerHTML = `
            <div>
                <button class="btn" style="margin-bottom: 2rem;" id="back-conv">
                    <i class="fa-solid fa-arrow-left"></i> Back
                </button>
            </div>
            <div style="max-width: 600px; margin: 0 auto; width: 100%;">
                <h2 style="margin-bottom: 1rem; text-align: center;">${conv.title}</h2>
                <div id="dialogue-container">
                    ${conv.dialogue.map((line, index) => `
                        <div class="dialogue-bubble ${index % 2 !== 0 ? 'alt' : ''}">
                            <div class="bubble-content">
                                <div class="bubble-speaker">${line.speaker}</div>
                                <div class="bubble-text">${line.text}</div>
                            </div>
                            <button class="btn sound-btn" onclick="window.speakTarget('${line.text.replace(/'/g, "\\'")}')">
                                <i class="fa-solid fa-volume-high" style="font-size: 0.8rem;"></i>
                            </button>
                        </div>
                    `).join('')}
                </div>
            </div>
        `;
        document.getElementById('back-conv').addEventListener('click', () => renderModule());
    }

    function renderListening(listeningData) {
        if (!listeningData || listeningData.length === 0) {
            contentArea.innerHTML = '<p>No listening exercises found.</p>';
            return;
        }

        let html = '<div style="display: flex; flex-direction: column; gap: 2rem;">';
        listeningData.forEach(list => {
            html += `
                <div class="glass-panel" style="padding: 2rem;">
                    <h3>${list.title}</h3>
                    <div style="margin: 1.5rem 0; display:flex; gap: 1rem; align-items: center;">
                        <button class="btn" onclick="window.speakTarget('${list.text.replace(/'/g, "\\'")}')">
                            <i class="fa-solid fa-play"></i> Play Audio
                        </button>
                        <span style="color: var(--text-muted); font-size: 0.9rem;">(Listen carefully to the English audio)</span>
                    </div>
                    
                    <div style="margin-top: 2rem;">
                        <h4 style="margin-bottom: 1.5rem; color: var(--accent-color); font-size: 1.2rem;">Questions:</h4>
                        ${list.questions.map((q, i) => `
                            <div style="margin-bottom: 2rem; background: rgba(0,0,0,0.1); padding: 1.5rem; border-radius: 16px; border: 1px solid var(--glass-border);">
                                <p style="font-size: 1.15rem; margin-bottom: 1rem; font-weight: 600;"><strong>${parseInt(i)+1}. ${q.q}</strong></p>
                                <div class="listening-options" style="display: flex; flex-direction: column; gap: 0.8rem;">
                                    ${q.options.map(opt => `
                                        <label>
                                            <input type="radio" name="q${list.id}_${i}" value="${opt}">
                                            <span>${opt}</span>
                                        </label>
                                    `).join('')}
                                </div>
                                <div style="margin-top: 1.5rem; display: flex; align-items: center; gap: 1rem;">
                                    <button class="btn" style="padding: 0.6rem 1.2rem; font-size: 0.9rem;" onclick="checkAnswer(this, '${q.answer}', 'q${list.id}_${i}')">Check Answer</button>
                                    <span class="feedback"></span>
                                </div>
                            </div>
                        `).join('')}
                    </div>
                </div>
            `;
        });
        html += '</div>';
        contentArea.innerHTML = html;

        // Add checkAnswer to window for inline access
        window.checkAnswer = function(btn, correctAnswer, groupName) {
            const selected = document.querySelector(`input[name="\${groupName}"]:checked`);
            const feedbackSpan = btn.nextElementSibling;
            if (!selected) {
                feedbackSpan.textContent = "Please select an answer.";
                feedbackSpan.style.color = "var(--text-muted)";
                return;
            }
            if (selected.value === correctAnswer) {
                feedbackSpan.textContent = "Correct! ✓";
                feedbackSpan.style.color = "var(--accent-color)";
            } else {
                feedbackSpan.textContent = "Incorrect. Try again ✗";
                feedbackSpan.style.color = "red";
            }
        };
    }

    function renderPhonetics(phoneticsData) {
        if (!phoneticsData || phoneticsData.length === 0) {
            contentArea.innerHTML = '<p>No phonetics data found.</p>';
            return;
        }

        let html = `
            <div>
                <p style="margin-bottom: 1.5rem; color: var(--text-muted);">Learn the International Phonetic Alphabet (IPA) and master your pronunciation.</p>
                <div style="display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 1.5rem;">
        `;
        phoneticsData.forEach(p => {
            html += `
                <div class="glass-panel" style="padding: 1.5rem; text-align: center;">
                    <div style="font-size: 3rem; font-weight: 800; color: var(--primary-color); margin-bottom: 1rem;">${p.sound}</div>
                    <p style="margin-bottom: 1rem;">Example: <strong>${p.word}</strong></p>
                    <p style="margin-bottom: 1.5rem; color: var(--text-muted); font-size: 0.9rem;">${p.text}</p>
                    <button class="btn" onclick="window.speakTarget('${p.word}')">
                        <i class="fa-solid fa-volume-high"></i> Pronounce
                    </button>
                </div>
            `;
        });
        html += '</div></div>';
        contentArea.innerHTML = html;
    }

    // Initial render
    renderModule();
});
