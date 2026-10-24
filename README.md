# examenes-generator
import React, { useState, useEffect, useMemo } from 'react';
import { 
  Folder, FolderPlus, FolderOpen, FileText, Plus, Trash2, Edit3, 
  Search, Play, Printer, CheckCircle, XCircle, Clock, Sparkles, 
  ChevronRight, ChevronDown, BookOpen, Layers, Award, RefreshCw, 
  Download, ArrowLeft, Check, HelpCircle, FileUp, Video, ListFilter,
  Eye, Save, AlertCircle, Sun, Moon, Sliders, ChevronDownCheck
} from 'lucide-react';

// --- INITIAL MOCK / PRESET DATA ---
const INITIAL_FOLDERS = [
  {
    id: 'course-1',
    name: '2º Bachillerato - Matemáticas II',
    parentId: null,
    type: 'course',
    children: [
      {
        id: 'term-1',
        name: '1º Trimestre',
        parentId: 'course-1',
        type: 'term',
        children: [
          { id: 'topic-1', name: 'Tema 1: Límites y Continuidad', parentId: 'term-1', type: 'topic' },
          { id: 'topic-2', name: 'Tema 2: Derivadas y Aplicaciones', parentId: 'term-1', type: 'topic' }
        ]
      },
      {
        id: 'term-2',
        name: '2º Trimestre',
        parentId: 'course-1',
        type: 'term',
        children: [
          { id: 'topic-3', name: 'Tema 3: Integrales Indefinidas', parentId: 'term-2', type: 'topic' }
        ]
      }
    ]
  },
  {
    id: 'course-2',
    name: '1º Bachillerato - Biología',
    parentId: null,
    type: 'course',
    children: [
      {
        id: 'term-bio-1',
        name: '1º Trimestre',
        parentId: 'course-2',
        type: 'term',
        children: [
          { id: 'topic-bio-1', name: 'Tema 1: La Célula y Orgánulos', parentId: 'term-bio-1', type: 'topic' }
        ]
      }
    ]
  }
];

const INITIAL_EXAMS = [
  {
    id: 'exam-demo-1',
    topicId: 'topic-2',
    title: 'Examen Diagnóstico: Derivadas y Regla de la Cadena',
    description: 'Evaluación rápida de conceptos de derivadas básicas y aplicaciones geométricas.',
    difficulty: 'Medium',
    createdAt: new Date().toISOString(),
    tags: ['Derivadas', 'Matemáticas', 'Examen'],
    questions: [
      {
        id: 'q1',
        type: 'multiple_choice',
        question: '¿Cuál es la derivada de f(x) = x³ - 4x + 7?',
        options: ['3x² - 4', '3x² + 4', 'x² - 4', '3x³ - 4'],
        correctAnswer: '3x² - 4',
        explanation: 'Aplicando la regla de la potencia: d/dx(x³) = 3x² y d/dx(-4x) = -4. La constante 7 se convierte en 0.'
      },
      {
        id: 'q2',
        type: 'true_false',
        question: 'Si una función es continua en un punto, siempre es derivable en dicho punto.',
        options: ['Verdadero', 'Falso'],
        correctAnswer: 'Falso',
        explanation: 'Ejemplo clásico: f(x) = |x| en x = 0 es continua pero no es derivable (tiene un pico).'
      },
      {
        id: 'q3',
        type: 'short_answer',
        question: '¿Qué representa geométricamente la primera derivada de una función en un punto?',
        correctAnswer: 'La pendiente de la recta tangente a la curva en ese punto.',
        explanation: 'La derivada es el límite del cociente incremental, representando la pendiente geométrica instantánea.'
      }
    ]
  }
];

export default function App() {
  // --- STATES ---
  const [folders, setFolders] = useState(() => {
    const saved = localStorage.getItem('exam_app_folders');
    return saved ? JSON.parse(saved) : INITIAL_FOLDERS;
  });

  const [exams, setExams] = useState(() => {
    const saved = localStorage.getItem('exam_app_exams');
    return saved ? JSON.parse(saved) : INITIAL_EXAMS;
  });

  const [attempts, setAttempts] = useState(() => {
    const saved = localStorage.getItem('exam_app_attempts');
    return saved ? JSON.parse(saved) : [];
  });

  // UI state
  const [activeTab, setActiveTab] = useState('generator'); // 'generator' | 'library' | 'taking' | 'print'
  const [selectedTopicId, setSelectedTopicId] = useState('topic-2');
  const [activeExam, setActiveExam] = useState(null);
  const [searchQuery, setSearchQuery] = useState('');
  const [filterDifficulty, setFilterDifficulty] = useState('all');
  const [darkMode, setDarkMode] = useState(false);

  // Folder creation/edit modal state
  const [modalOpen, setModalOpen] = useState(false);
  const [modalMode, setModalMode] = useState('create'); // 'create' | 'edit'
  const [targetParentFolder, setTargetParentFolder] = useState(null);
  const [editingFolder, setEditingFolder] = useState(null);
  const [folderFormName, setFolderFormName] = useState('');
  const [folderFormType, setFolderFormType] = useState('course'); // 'course' | 'term' | 'topic'

  // Generator State
  const [sourceType, setSourceType] = useState('text'); // 'text' | 'file' | 'video'
  const [sourceContent, setSourceContent] = useState('');
  const [videoUrl, setVideoUrl] = useState('');
  const [numQuestions, setNumQuestions] = useState(5);
  const [difficulty, setDifficulty] = useState('Medium');
  const [questionTypes, setQuestionTypes] = useState({
    multiple_choice: true,
    true_false: true,
    short_answer: true,
    essay: false
  });
  const [examTitle, setExamTitle] = useState('');
  const [isGenerating, setIsGenerating] = useState(false);
  const [generationError, setGenerationError] = useState('');

  // Exam Taking State
  const [userAnswers, setUserAnswers] = useState({});
  const [timeRemaining, setTimeRemaining] = useState(1800); // 30 min default
  const [isExamSubmitted, setIsExamSubmitted] = useState(false);
  const [examScore, setExamScore] = useState(null);

  // Print Mode state
  const [showAnswerKey, setShowAnswerKey] = useState(true);

  // Save to local storage
  useEffect(() => {
    localStorage.setItem('exam_app_folders', JSON.stringify(folders));
  }, [folders]);

  useEffect(() => {
    localStorage.setItem('exam_app_exams', JSON.stringify(exams));
  }, [exams]);

  useEffect(() => {
    localStorage.setItem('exam_app_attempts', JSON.stringify(attempts));
  }, [attempts]);

  // Timer Effect during exam taking
  useEffect(() => {
    let timer;
    if (activeTab === 'taking' && !isExamSubmitted && timeRemaining > 0) {
      timer = setInterval(() => {
        setTimeRemaining(prev => {
          if (prev <= 1) {
            handleFinalizeExam();
            return 0;
          }
          return prev - 1;
        });
      }, 1000);
    }
    return () => clearInterval(timer);
  }, [activeTab, isExamSubmitted, timeRemaining]);

  // --- HELPER FUNCTIONS FOR TREE ---
  const findFolderById = (nodes, id) => {
    for (const node of nodes) {
      if (node.id === id) return node;
      if (node.children) {
        const found = findFolderById(node.children, id);
        if (found) return found;
      }
    }
    return null;
  };

  const addFolderToTree = (nodes, parentId, newFolder) => {
    if (!parentId) return [...nodes, newFolder];
    return nodes.map(node => {
      if (node.id === parentId) {
        return {
          ...node,
          children: [...(node.children || []), newFolder]
        };
      }
      if (node.children) {
        return {
          ...node,
          children: addFolderToTree(node.children, parentId, newFolder)
        };
      }
      return node;
    });
  };

  const deleteFolderFromTree = (nodes, id) => {
    return nodes.filter(node => node.id !== id).map(node => {
      if (node.children) {
        return {
          ...node,
          children: deleteFolderFromTree(node.children, id)
        };
      }
      return node;
    });
  };

  const updateFolderNameInTree = (nodes, id, newName) => {
    return nodes.map(node => {
      if (node.id === id) {
        return { ...node, name: newName };
      }
      if (node.children) {
        return {
          ...node,
          children: updateFolderNameInTree(node.children, id, newName)
        };
      }
      return node;
    });
  };

  // Get flat list of topics for dropdown selections
  const getAllTopics = (nodes) => {
    let result = [];
    nodes.forEach(node => {
      if (node.type === 'topic') {
        result.push(node);
      }
      if (node.children) {
        result = result.concat(getAllTopics(node.children));
      }
    });
    return result;
  };

  const allTopics = useMemo(() => getAllTopics(folders), [folders]);

  // --- HANDLERS: FOLDERS ---
  const handleOpenCreateModal = (parent = null, defaultType = 'course') => {
    setModalMode('create');
    setTargetParentFolder(parent);
    setFolderFormType(defaultType);
    setFolderFormName('');
    setModalOpen(true);
  };

  const handleOpenEditModal = (folder) => {
    setModalMode('edit');
    setEditingFolder(folder);
    setFolderFormName(folder.name);
    setModalOpen(true);
  };

  const handleSaveFolder = (e) => {
    e.preventDefault();
    if (!folderFormName.trim()) return;

    if (modalMode === 'create') {
      const newFolder = {
        id: 'f_' + Date.now(),
        name: folderFormName.trim(),
        parentId: targetParentFolder ? targetParentFolder.id : null,
        type: folderFormType,
        children: folderFormType !== 'topic' ? [] : undefined
      };
      setFolders(prev => addFolderToTree(prev, targetParentFolder?.id, newFolder));
    } else if (modalMode === 'edit' && editingFolder) {
      setFolders(prev => updateFolderNameInTree(prev, editingFolder.id, folderFormName.trim()));
    }

    setModalOpen(false);
  };

  const handleDeleteFolder = (id) => {
    if (window.confirm('¿Estás seguro de eliminar esta carpeta y todo su contenido?')) {
      setFolders(prev => deleteFolderFromTree(prev, id));
      // Delete exams in this topic
      setExams(prev => prev.filter(e => e.topicId !== id));
    }
  };

  // --- HANDLERS: EXAM GENERATION WITH GEMINI API ---
  const handleFileUpload = (e) => {
    const file = e.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (evt) => {
      setSourceContent(evt.target.result);
    };
    reader.readAsText(file);
  };

  const generateExamWithGemini = async () => {
    let promptContent = sourceContent;
    if (sourceType === 'video') {
      promptContent = `Transcripción o resumen del video/enlace (${videoUrl}):\n${sourceContent}`;
    }

    if (!promptContent.trim()) {
      setGenerationError('Por favor proporciona texto, sube un archivo o escribe las notas del contenido para generar el examen.');
      return;
    }

    setIsGenerating(true);
    setGenerationError('');

    const selectedTypes = Object.keys(questionTypes).filter(k => questionTypes[k]);
    if (selectedTypes.length === 0) {
      setGenerationError('Selecciona al menos un tipo de pregunta.');
      setIsGenerating(false);
      return;
    }

    const systemPrompt = `Eres un profesor experto creando evaluaciones educativas en español. 
Genera un examen estructurado en formato JSON con las siguientes pautas:
- Número de preguntas: ${numQuestions}
- Dificultad: ${difficulty}
- Tipos de preguntas permitidos: ${selectedTypes.join(', ')}
- El JSON retornado DEBE ser un arreglo válido de objetos pregunta sin texto o bloques markdown alrededor, con la siguiente estructura:
[
  {
    "id": "q1",
    "type": "multiple_choice" | "true_false" | "short_answer" | "essay",
    "question": "Texto de la pregunta...",
    "options": ["Opción A", "Opción B", "Opción C", "Opción D"], // Solo si es tipo multiple_choice o true_false
    "correctAnswer": "Opción correcta exacta o respuesta esperada",
    "explanation": "Explicación clara y detallada de la respuesta correcta"
  }
]`;

    const userQuery = `Basándote estricta y detalladamente en el siguiente texto de estudio, genera las ${numQuestions} preguntas:

=== INICIO DEL TEXTO ===
${promptContent.substring(0, 10000)}
=== FIN DEL TEXTO ===`;

    try {
      const apiKey = "";
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

      const payload = {
        contents: [{ parts: [{ text: userQuery }] }],
        systemInstruction: { parts: [{ text: systemPrompt }] },
        generationConfig: {
          responseMimeType: "application/json"
        }
      };

      const res = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      if (!res.ok) {
        throw new Error(`Error en la API de Gemini: ${res.statusText}`);
      }

      const data = await res.json();
      const rawText = data.candidates?.[0]?.content?.parts?.[0]?.text;

      if (!rawText) {
        throw new Error('No se recibió contenido válido de la IA.');
      }

      const parsedQuestions = JSON.parse(rawText);

      const targetTopic = allTopics.find(t => t.id === selectedTopicId);
      const newExam = {
        id: 'exam_' + Date.now(),
        topicId: selectedTopicId || (allTopics[0]?.id || 'uncategorized'),
        title: examTitle.trim() || `Examen de ${targetTopic ? targetTopic.name : 'Tema Seleccionado'}`,
        description: `Examen generado automáticamente (${difficulty}) con ${parsedQuestions.length} preguntas.`,
        difficulty: difficulty,
        createdAt: new Date().toISOString(),
        tags: [difficulty, 'IA Generada'],
        questions: parsedQuestions
      };

      setExams(prev => [newExam, ...prev]);
      setActiveExam(newExam);
      setActiveTab('library');
      setIsGenerating(false);

      // Reset Form
      setSourceContent('');
      setExamTitle('');
    } catch (err) {
      console.error(err);
      setGenerationError(`Error al generar el examen: ${err.message}. Asegúrate de que el texto sea claro.`);
      setIsGenerating(false);
    }
  };

  // --- HANDLERS: EXAM TAKING & GRADING ---
  const handleStartExam = (exam) => {
    setActiveExam(exam);
    setUserAnswers({});
    setTimeRemaining(exam.questions.length * 120); // 2 mins per question
    setIsExamSubmitted(false);
    setExamScore(null);
    setActiveTab('taking');
  };

  const handleAnswerSelect = (qId, answer) => {
    setUserAnswers(prev => ({ ...prev, [qId]: answer }));
  };

  const handleFinalizeExam = () => {
    if (!activeExam) return;

    let correctCount = 0;
    let totalScorable = 0;

    activeExam.questions.forEach(q => {
      if (q.type === 'multiple_choice' || q.type === 'true_false') {
        totalScorable++;
        const userAns = (userAnswers[q.id] || '').trim().toLowerCase();
        const correctAns = (q.correctAnswer || '').trim().toLowerCase();
        if (userAns === correctAns) {
          correctCount++;
        }
      }
    });

    const scorePct = totalScorable > 0 ? Math.round((correctCount / totalScorable) * 100) : 100;
    
    setExamScore({
      correct: correctCount,
      total: totalScorable,
      percentage: scorePct
    });

    setIsExamSubmitted(true);

    // Save attempt record
    const attemptRecord = {
      id: 'att_' + Date.now(),
      examId: activeExam.id,
      examTitle: activeExam.title,
      score: scorePct,
      date: new Date().toISOString()
    };
    setAttempts(prev => [attemptRecord, ...prev]);
  };

  // --- FILTERED EXAMS FOR LIBRARY ---
  const filteredExams = useMemo(() => {
    return exams.filter(e => {
      const matchesSearch = e.title.toLowerCase().includes(searchQuery.toLowerCase()) ||
                            e.description.toLowerCase().includes(searchQuery.toLowerCase());
      const matchesFolder = selectedTopicId ? e.topicId === selectedTopicId : true;
      const matchesDifficulty = filterDifficulty === 'all' ? true : e.difficulty === filterDifficulty;
      return matchesSearch && matchesFolder && matchesDifficulty;
    });
  }, [exams, searchQuery, selectedTopicId, filterDifficulty]);

  // Format seconds to MM:SS
  const formatTime = (secs) => {
    const m = Math.floor(secs / 60);
    const s = secs % 60;
    return `${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`;
  };

  return (
    <div className={`min-h-screen font-sans ${darkMode ? 'bg-slate-900 text-slate-100' : 'bg-slate-50 text-slate-800'} flex flex-col md:flex-row transition-colors duration-200`}>
      
      {/* --- SIDEBAR: HIERARCHY / FOLDER TREE --- */}
      <aside className={`w-full md:w-80 border-r ${darkMode ? 'bg-slate-800/80 border-slate-700' : 'bg-white border-slate-200'} p-4 flex flex-col flex-shrink-0`}>
        <div className="flex items-center justify-between mb-6 pb-3 border-b border-slate-200 dark:border-slate-700">
          <div className="flex items-center gap-2">
            <div className="p-2 bg-indigo-600 text-white rounded-lg shadow">
              <Sparkles className="w-5 h-5" />
            </div>
            <div>
              <h1 className="font-bold text-lg leading-tight text-slate-900 dark:text-white">ExamAI Studio</h1>
              <p className="text-xs text-slate-500 dark:text-slate-400">Generador y Organizador</p>
            </div>
          </div>
          <button 
            onClick={() => setDarkMode(!darkMode)}
            className="p-1.5 rounded-lg text-slate-500 hover:bg-slate-100 dark:hover:bg-slate-700 dark:text-slate-400"
            title="Cambiar Modo Oscuro/Claro"
          >
            {darkMode ? <Sun className="w-4 h-4 text-amber-400" /> : <Moon className="w-4 h-4" />}
          </button>
        </div>

        {/* Action: New Course */}
        <div className="mb-4 flex gap-2">
          <button
            onClick={() => handleOpenCreateModal(null, 'course')}
            className="w-full py-2 px-3 bg-indigo-600 hover:bg-indigo-700 text-white rounded-lg text-sm font-medium flex items-center justify-center gap-2 shadow-sm transition-all"
          >
            <FolderPlus className="w-4 h-4" />
            Nuevo Curso
          </button>
        </div>

        {/* Folder Tree Title */}
        <div className="flex items-center justify-between text-xs font-semibold uppercase tracking-wider text-slate-400 mb-2 px-1">
          <span>Estructura de Cursos</span>
          <button 
            onClick={() => setSelectedTopicId(null)}
            className={`hover:underline text-indigo-500 text-xs ${!selectedTopicId ? 'font-bold' : ''}`}
          >
            Ver todos
          </button>
        </div>

        {/* Folder Hierarchy List */}
        <div className="flex-1 overflow-y-auto space-y-1 pr-1 custom-scrollbar">
          {folders.length === 0 ? (
            <p className="text-xs text-slate-400 italic p-2">No hay cursos creados.</p>
          ) : (
            folders.map(course => (
              <FolderNode 
                key={course.id} 
                node={course} 
                selectedTopicId={selectedTopicId}
                setSelectedTopicId={setSelectedTopicId}
                onAddSubfolder={handleOpenCreateModal}
                onEditFolder={handleOpenEditModal}
                onDeleteFolder={handleDeleteFolder}
                darkMode={darkMode}
              />
            ))
          )}
        </div>

        {/* Quick Stats / Footer */}
        <div className="mt-auto pt-4 border-t border-slate-200 dark:border-slate-700 text-xs text-slate-500 dark:text-slate-400 flex justify-between">
          <span>Total Exámenes: <strong>{exams.length}</strong></span>
          <span>Intentos: <strong>{attempts.length}</strong></span>
        </div>
      </aside>

      {/* --- MAIN CONTENT AREA --- */}
      <main className="flex-1 flex flex-col min-w-0 overflow-y-auto">
        
        {/* Navigation Bar */}
        <header className={`border-b ${darkMode ? 'bg-slate-800/50 border-slate-700' : 'bg-white border-slate-200'} px-6 py-4 flex flex-wrap items-center justify-between gap-4 sticky top-0 z-10 backdrop-blur-md`}>
          <div className="flex items-center gap-2">
            <button
              onClick={() => setActiveTab('generator')}
              className={`px-4 py-2 rounded-lg text-sm font-medium flex items-center gap-2 transition-all ${
                activeTab === 'generator'
                  ? 'bg-indigo-600 text-white shadow-sm'
                  : 'text-slate-600 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-700'
              }`}
            >
              <Sparkles className="w-4 h-4" />
              Generar Examen (IA)
            </button>

            <button
              onClick={() => setActiveTab('library')}
              className={`px-4 py-2 rounded-lg text-sm font-medium flex items-center gap-2 transition-all ${
                activeTab === 'library'
                  ? 'bg-indigo-600 text-white shadow-sm'
                  : 'text-slate-600 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-700'
              }`}
            >
              <BookOpen className="w-4 h-4" />
              Biblioteca
            </button>
          </div>

          {/* Current Selection Breadcrumb Info */}
          <div className="text-xs text-slate-500 dark:text-slate-400 flex items-center gap-1">
            <span>Filtro activo:</span>
            <span className="font-semibold text-indigo-600 dark:text-indigo-400">
              {selectedTopicId 
                ? (allTopics.find(t => t.id === selectedTopicId)?.name || 'Tema no encontrado') 
                : 'Todos los Temas'}
            </span>
          </div>
        </header>

        {/* TAB 1: GENERATOR MODE */}
        {activeTab === 'generator' && (
          <div className="p-6 max-w-4xl mx-auto w-full space-y-6">
            <div className="border-b border-slate-200 dark:border-slate-700 pb-4">
              <h2 className="text-2xl font-bold text-slate-900 dark:text-white flex items-center gap-2">
                <Sparkles className="w-6 h-6 text-indigo-600" />
                Generador Automático de Exámenes
              </h2>
              <p className="text-sm text-slate-500 dark:text-slate-400 mt-1">
                Adjunta texto, apuntes o resúmenes de video para que la IA estructurada genere un examen listo para evaluar.
              </p>
            </div>

            {generationError && (
              <div className="p-4 bg-red-50 dark:bg-red-900/30 border border-red-200 dark:border-red-800 text-red-700 dark:text-red-300 rounded-lg text-sm flex items-start gap-2">
                <AlertCircle className="w-5 h-5 flex-shrink-0 mt-0.5" />
                <div>{generationError}</div>
              </div>
            )}

            {/* Config Box 1: Topic Destination */}
            <div className={`p-5 rounded-xl border ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-white border-slate-200'} shadow-sm space-y-4`}>
              <h3 className="text-sm font-bold uppercase tracking-wider text-slate-500 dark:text-slate-400 flex items-center gap-2">
                <Folder className="w-4 h-4" /> 1. Destino y Clasificación
              </h3>

              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1">
                    Guardar en el Tema / Subcarpeta:
                  </label>
                  <select
                    value={selectedTopicId || ''}
                    onChange={(e) => setSelectedTopicId(e.target.value)}
                    className={`w-full p-2.5 rounded-lg border text-sm ${darkMode ? 'bg-slate-700 border-slate-600 text-white' : 'bg-slate-50 border-slate-300 text-slate-800'}`}
                  >
                    <option value="">-- Seleccionar Tema --</option>
                    {allTopics.map(t => (
                      <option key={t.id} value={t.id}>{t.name}</option>
                    ))}
                  </select>
                </div>

                <div>
                  <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1">
                    Título Personalizado del Examen:
                  </label>
                  <input
                    type="text"
                    placeholder="Ej: Examen Parcial de Derivadas y Límites"
                    value={examTitle}
                    onChange={(e) => setExamTitle(e.target.value)}
                    className={`w-full p-2.5 rounded-lg border text-sm ${darkMode ? 'bg-slate-700 border-slate-600 text-white' : 'bg-slate-50 border-slate-300 text-slate-800'}`}
                  />
                </div>
              </div>
            </div>

            {/* Config Box 2: Source Input */}
            <div className={`p-5 rounded-xl border ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-white border-slate-200'} shadow-sm space-y-4`}>
              <h3 className="text-sm font-bold uppercase tracking-wider text-slate-500 dark:text-slate-400 flex items-center gap-2">
                <FileText className="w-4 h-4" /> 2. Fuente del Contenido
              </h3>

              {/* Input Type Selector */}
              <div className="flex gap-2 border-b border-slate-200 dark:border-slate-700 pb-3">
                <button
                  type="button"
                  onClick={() => setSourceType('text')}
                  className={`px-3 py-1.5 rounded-lg text-xs font-medium flex items-center gap-1.5 transition-all ${
                    sourceType === 'text'
                      ? 'bg-indigo-100 text-indigo-700 dark:bg-indigo-900/50 dark:text-indigo-300'
                      : 'text-slate-500 hover:bg-slate-100 dark:hover:bg-slate-700'
                  }`}
                >
                  <Edit3 className="w-3.5 h-3.5" />
                  Texto / Apuntes
                </button>
                <button
                  type="button"
                  onClick={() => setSourceType('file')}
                  className={`px-3 py-1.5 rounded-lg text-xs font-medium flex items-center gap-1.5 transition-all ${
                    sourceType === 'file'
                      ? 'bg-indigo-100 text-indigo-700 dark:bg-indigo-900/50 dark:text-indigo-300'
                      : 'text-slate-500 hover:bg-slate-100 dark:hover:bg-slate-700'
                  }`}
                >
                  <FileUp className="w-3.5 h-3.5" />
                  Subir Archivo (.txt, .md)
                </button>
                <button
                  type="button"
                  onClick={() => setSourceType('video')}
                  className={`px-3 py-1.5 rounded-lg text-xs font-medium flex items-center gap-1.5 transition-all ${
                    sourceType === 'video'
                      ? 'bg-indigo-100 text-indigo-700 dark:bg-indigo-900/50 dark:text-indigo-300'
                      : 'text-slate-500 hover:bg-slate-100 dark:hover:bg-slate-700'
                  }`}
                >
                  <Video className="w-3.5 h-3.5" />
                  Transcripción de Video
                </button>
              </div>

              {sourceType === 'text' && (
                <div>
                  <textarea
                    rows={6}
                    placeholder="Pega aquí el contenido de tus lecciones, libros de texto o notas para que la IA extraiga el examen..."
                    value={sourceContent}
                    onChange={(e) => setSourceContent(e.target.value)}
                    className={`w-full p-3 rounded-lg border text-sm ${darkMode ? 'bg-slate-700 border-slate-600 text-white' : 'bg-slate-50 border-slate-300 text-slate-800'}`}
                  />
                </div>
              )}

              {sourceType === 'file' && (
                <div className="space-y-3">
                  <input
                    type="file"
                    accept=".txt,.md,.csv,.json"
                    onChange={handleFileUpload}
                    className="block w-full text-sm text-slate-500 file:mr-4 file:py-2 file:px-4 file:rounded-lg file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100 dark:file:bg-indigo-950 dark:file:text-indigo-300"
                  />
                  {sourceContent && (
                    <div className="p-3 bg-slate-100 dark:bg-slate-700 rounded-lg text-xs max-h-32 overflow-y-auto">
                      <strong>Vista previa del archivo:</strong>
                      <p className="mt-1 whitespace-pre-wrap">{sourceContent.substring(0, 300)}...</p>
                    </div>
                  )}
                </div>
              )}

              {sourceType === 'video' && (
                <div className="space-y-3">
                  <input
                    type="url"
                    placeholder="Enlace o URL del video (ej. YouTube)..."
                    value={videoUrl}
                    onChange={(e) => setVideoUrl(e.target.value)}
                    className={`w-full p-2.5 rounded-lg border text-sm ${darkMode ? 'bg-slate-700 border-slate-600 text-white' : 'bg-slate-50 border-slate-300 text-slate-800'}`}
                  />
                  <textarea
                    rows={4}
                    placeholder="Pega aquí la transcripción o los puntos clave explicados en el video..."
                    value={sourceContent}
                    onChange={(e) => setSourceContent(e.target.value)}
                    className={`w-full p-3 rounded-lg border text-sm ${darkMode ? 'bg-slate-700 border-slate-600 text-white' : 'bg-slate-50 border-slate-300 text-slate-800'}`}
                  />
                </div>
              )}
            </div>

            {/* Config Box 3: Exam Parameters */}
            <div className={`p-5 rounded-xl border ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-white border-slate-200'} shadow-sm space-y-4`}>
              <h3 className="text-sm font-bold uppercase tracking-wider text-slate-500 dark:text-slate-400 flex items-center gap-2">
                <Sliders className="w-4 h-4" /> 3. Parámetros del Examen
              </h3>

              <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div>
                  <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-2">
                    Tipos de Preguntas Incluidas:
                  </label>
                  <div className="space-y-2">
                    {[
                      { id: 'multiple_choice', label: 'Opción Múltiple (4 alternativas)' },
                      { id: 'true_false', label: 'Verdadero / Falso' },
                      { id: 'short_answer', label: 'Respuesta Corta' },
                      { id: 'essay', label: 'Desarrollo / Ensayo' }
                    ].map(item => (
                      <label key={item.id} className="flex items-center gap-2 text-sm text-slate-700 dark:text-slate-300 cursor-pointer">
                        <input
                          type="checkbox"
                          checked={questionTypes[item.id]}
                          onChange={(e) => setQuestionTypes({ ...questionTypes, [item.id]: e.target.checked })}
                          className="rounded text-indigo-600 focus:ring-indigo-500"
                        />
                        {item.label}
                      </label>
                    ))}
                  </div>
                </div>

                <div className="space-y-4">
                  <div>
                    <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1">
                      Número de Preguntas: ({numQuestions})
                    </label>
                    <input
                      type="range"
                      min={3}
                      max={15}
                      value={numQuestions}
                      onChange={(e) => setNumQuestions(Number(e.target.value))}
                      className="w-full accent-indigo-600"
                    />
                  </div>

                  <div>
                    <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1">
                      Nivel de Dificultad:
                    </label>
                    <div className="flex gap-2">
                      {['Easy', 'Medium', 'Hard'].map(level => (
                        <button
                          key={level}
                          type="button"
                          onClick={() => setDifficulty(level)}
                          className={`flex-1 py-1.5 rounded-lg text-xs font-semibold border transition-all ${
                            difficulty === level
                              ? 'bg-indigo-600 text-white border-indigo-600 shadow-sm'
                              : darkMode
                              ? 'bg-slate-700 border-slate-600 text-slate-300'
                              : 'bg-white border-slate-300 text-slate-700'
                          }`}
                        >
                          {level === 'Easy' ? 'Fácil' : level === 'Medium' ? 'Medio' : 'Difícil'}
                        </button>
                      ))}
                    </div>
                  </div>
                </div>
              </div>
            </div>

            {/* Action Button */}
            <div className="pt-2">
              <button
                type="button"
                onClick={generateExamWithGemini}
                disabled={isGenerating}
                className="w-full py-3.5 bg-gradient-to-r from-indigo-600 to-violet-600 hover:from-indigo-700 hover:to-violet-700 text-white rounded-xl font-bold shadow-md flex items-center justify-center gap-2 transition-all disabled:opacity-50"
              >
                {isGenerating ? (
                  <>
                    <RefreshCw className="w-5 h-5 animate-spin" />
                    Generando Examen con IA Gemini...
                  </>
                ) : (
                  <>
                    <Sparkles className="w-5 h-5" />
                    Generar Examen Ahora
                  </>
                )}
              </button>
            </div>
          </div>
        )}

        {/* TAB 2: EXAM LIBRARY */}
        {activeTab === 'library' && (
          <div className="p-6 max-w-5xl mx-auto w-full space-y-6">
            <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 border-b border-slate-200 dark:border-slate-700 pb-4">
              <div>
                <h2 className="text-2xl font-bold text-slate-900 dark:text-white flex items-center gap-2">
                  <BookOpen className="w-6 h-6 text-indigo-600" />
                  Biblioteca de Exámenes
                </h2>
                <p className="text-sm text-slate-500 dark:text-slate-400 mt-1">
                  Exámenes guardados y clasificados por temas.
                </p>
              </div>

              {/* Search & Filters */}
              <div className="flex flex-wrap items-center gap-2">
                <div className="relative">
                  <Search className="w-4 h-4 absolute left-3 top-2.5 text-slate-400" />
                  <input
                    type="text"
                    placeholder="Buscar examen..."
                    value={searchQuery}
                    onChange={(e) => setSearchQuery(e.target.value)}
                    className={`pl-9 pr-3 py-1.5 rounded-lg border text-sm ${darkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-white border-slate-300'}`}
                  />
                </div>

                <select
                  value={filterDifficulty}
                  onChange={(e) => setFilterDifficulty(e.target.value)}
                  className={`p-1.5 rounded-lg border text-sm ${darkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-white border-slate-300'}`}
                >
                  <option value="all">Toda dificultad</option>
                  <option value="Easy">Fácil</option>
                  <option value="Medium">Medio</option>
                  <option value="Hard">Difícil</option>
                </select>
              </div>
            </div>

            {filteredExams.length === 0 ? (
              <div className="text-center py-12 border-2 border-dashed border-slate-200 dark:border-slate-700 rounded-xl p-6">
                <FileText className="w-12 h-12 text-slate-300 dark:text-slate-600 mx-auto mb-3" />
                <h3 className="text-base font-semibold text-slate-700 dark:text-slate-300">No hay exámenes en este tema</h3>
                <p className="text-xs text-slate-400 mt-1 max-w-sm mx-auto">
                  Selecciona otro tema en el panel lateral o usa el generador IA para crear uno nuevo.
                </p>
                <button
                  onClick={() => setActiveTab('generator')}
                  className="mt-4 px-4 py-2 bg-indigo-600 text-white rounded-lg text-xs font-semibold hover:bg-indigo-700 transition-all"
                >
                  Generar Primer Examen
                </button>
              </div>
            ) : (
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                {filteredExams.map(exam => {
                  const topicObj = allTopics.find(t => t.id === exam.topicId);
                  return (
                    <div 
                      key={exam.id}
                      className={`p-5 rounded-xl border ${darkMode ? 'bg-slate-800 border-slate-700 hover:border-indigo-500' : 'bg-white border-slate-200 hover:border-indigo-300'} shadow-sm transition-all flex flex-col justify-between`}
                    >
                      <div>
                        <div className="flex items-start justify-between gap-2 mb-2">
                          <span className="px-2.5 py-0.5 rounded-full text-xs font-semibold bg-indigo-50 text-indigo-700 dark:bg-indigo-950 dark:text-indigo-300">
                            {topicObj ? topicObj.name : 'Tema General'}
                          </span>
                          <span className={`px-2 py-0.5 rounded text-[10px] font-bold uppercase ${
                            exam.difficulty === 'Easy' ? 'bg-emerald-100 text-emerald-700 dark:bg-emerald-950 dark:text-emerald-300' :
                            exam.difficulty === 'Hard' ? 'bg-rose-100 text-rose-700 dark:bg-rose-950 dark:text-rose-300' :
                            'bg-amber-100 text-amber-700 dark:bg-amber-950 dark:text-amber-300'
                          }`}>
                            {exam.difficulty}
                          </span>
                        </div>

                        <h3 className="font-bold text-slate-900 dark:text-white text-base mb-1">{exam.title}</h3>
                        <p className="text-xs text-slate-500 dark:text-slate-400 mb-3">{exam.description}</p>
                        
                        <div className="flex items-center gap-4 text-xs text-slate-400 mb-4">
                          <span>{exam.questions.length} Preguntas</span>
                          <span>•</span>
                          <span>{new Date(exam.createdAt).toLocaleDateString()}</span>
                        </div>
                      </div>

                      {/* Action buttons */}
                      <div className="flex items-center gap-2 pt-3 border-t border-slate-100 dark:border-slate-700/50">
                        <button
                          onClick={() => handleStartExam(exam)}
                          className="flex-1 py-2 px-3 bg-indigo-600 hover:bg-indigo-700 text-white rounded-lg text-xs font-bold flex items-center justify-center gap-1.5 transition-all"
                        >
                          <Play className="w-3.5 h-3.5" />
                          Realizar
                        </button>
                        <button
                          onClick={() => {
                            setActiveExam(exam);
                            setActiveTab('print');
                          }}
                          className={`py-2 px-3 rounded-lg text-xs font-medium border flex items-center gap-1.5 transition-all ${
                            darkMode 
                              ? 'border-slate-600 hover:bg-slate-700 text-slate-300' 
                              : 'border-slate-300 hover:bg-slate-100 text-slate-700'
                          }`}
                        >
                          <Printer className="w-3.5 h-3.5" />
                          Imprimir
                        </button>
                        <button
                          onClick={() => {
                            if (window.confirm('¿Eliminar este examen?')) {
                              setExams(prev => prev.filter(e => e.id !== exam.id));
                            }
                          }}
                          className="p-2 text-red-500 hover:bg-red-50 dark:hover:bg-red-950/50 rounded-lg transition-all"
                          title="Eliminar Examen"
                        >
                          <Trash2 className="w-3.5 h-3.5" />
                        </button>
                      </div>
                    </div>
                  );
                })}
              </div>
            )}
          </div>
        )}

        {/* TAB 3: EXAM TAKING MODE */}
        {activeTab === 'taking' && activeExam && (
          <div className="p-6 max-w-3xl mx-auto w-full space-y-6">
            <div className="flex items-center justify-between">
              <button
                onClick={() => setActiveTab('library')}
                className="text-xs font-semibold text-indigo-600 dark:text-indigo-400 flex items-center gap-1 hover:underline"
              >
                <ArrowLeft className="w-4 h-4" /> Volver a la Biblioteca
              </button>

              {!isExamSubmitted && (
                <div className="flex items-center gap-2 bg-amber-50 dark:bg-amber-950/50 text-amber-700 dark:text-amber-300 px-3 py-1.5 rounded-full text-xs font-bold border border-amber-200 dark:border-amber-800">
                  <Clock className="w-4 h-4 animate-pulse" />
                  Tiempo Restante: {formatTime(timeRemaining)}
                </div>
              )}
            </div>

            {/* Exam Header Banner */}
            <div className={`p-6 rounded-xl border ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-white border-slate-200'} shadow-sm`}>
              <h2 className="text-xl font-bold text-slate-900 dark:text-white mb-2">{activeExam.title}</h2>
              <p className="text-xs text-slate-500 dark:text-slate-400">{activeExam.description}</p>
            </div>

            {/* Results Card if Submitted */}
            {isExamSubmitted && examScore && (
              <div className="p-6 bg-gradient-to-br from-indigo-500 to-purple-600 text-white rounded-xl shadow-md space-y-2 text-center">
                <Award className="w-12 h-12 mx-auto text-amber-300" />
                <h3 className="text-xl font-bold">¡Examen Finalizado!</h3>
                <p className="text-3xl font-black">{examScore.percentage}%</p>
                <p className="text-xs opacity-90">
                  Respondiste correctamente {examScore.correct} de {examScore.total} preguntas de selección múltiple.
                </p>
              </div>
            )}

            {/* Questions List */}
            <div className="space-y-6">
              {activeExam.questions.map((q, idx) => (
                <div 
                  key={q.id || idx} 
                  className={`p-6 rounded-xl border ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-white border-slate-200'} shadow-sm space-y-4`}
                >
                  <div className="flex items-start justify-between gap-2">
                    <span className="font-bold text-indigo-600 dark:text-indigo-400 text-sm">
                      Pregunta {idx + 1}
                    </span>
                    <span className="text-[10px] uppercase font-bold text-slate-400 bg-slate-100 dark:bg-slate-700 px-2 py-0.5 rounded">
                      {q.type.replace('_', ' ')}
                    </span>
                  </div>

                  <p className="font-medium text-slate-800 dark:text-slate-100 text-base">{q.question}</p>

                  {/* Options for Multiple Choice or True/False */}
                  {(q.type === 'multiple_choice' || q.type === 'true_false') && (
                    <div className="space-y-2 pt-2">
                      {q.options?.map((opt, oIdx) => {
                        const isSelected = userAnswers[q.id] === opt;
                        const isCorrect = q.correctAnswer === opt;
                        let optionStyle = darkMode ? 'bg-slate-700/50 border-slate-600' : 'bg-slate-50 border-slate-200';

                        if (isExamSubmitted) {
                          if (isCorrect) {
                            optionStyle = 'bg-emerald-100 dark:bg-emerald-950 border-emerald-400 text-emerald-800 dark:text-emerald-200 font-semibold';
                          } else if (isSelected && !isCorrect) {
                            optionStyle = 'bg-rose-100 dark:bg-rose-950 border-rose-400 text-rose-800 dark:text-rose-200';
                          }
                        } else if (isSelected) {
                          optionStyle = 'bg-indigo-50 dark:bg-indigo-950 border-indigo-500 text-indigo-700 dark:text-indigo-300 font-medium';
                        }

                        return (
                          <button
                            key={oIdx}
                            disabled={isExamSubmitted}
                            onClick={() => handleAnswerSelect(q.id, opt)}
                            className={`w-full text-left p-3 rounded-lg border text-sm transition-all flex items-center justify-between ${optionStyle}`}
                          >
                            <span>{opt}</span>
                            {isExamSubmitted && isCorrect && <Check className="w-4 h-4 text-emerald-600" />}
                            {isExamSubmitted && isSelected && !isCorrect && <XCircle className="w-4 h-4 text-rose-600" />}
                          </button>
                        );
                      })}
                    </div>
                  )}

                  {/* Short Answer or Essay */}
                  {(q.type === 'short_answer' || q.type === 'essay') && (
                    <div className="pt-2">
                      <textarea
                        rows={q.type === 'essay' ? 4 : 2}
                        disabled={isExamSubmitted}
                        placeholder="Escribe tu respuesta aquí..."
                        value={userAnswers[q.id] || ''}
                        onChange={(e) => handleAnswerSelect(q.id, e.target.value)}
                        className={`w-full p-3 rounded-lg border text-sm ${darkMode ? 'bg-slate-700 border-slate-600 text-white' : 'bg-slate-50 border-slate-300 text-slate-800'}`}
                      />
                    </div>
                  )}

                  {/* Explanation Block post submission */}
                  {isExamSubmitted && (
                    <div className="p-3 bg-amber-50 dark:bg-amber-950/40 border-l-4 border-amber-400 rounded-r-lg text-xs space-y-1 text-amber-900 dark:text-amber-200">
                      <p className="font-bold">Pauta de Corrección & Explicación:</p>
                      <p><strong>Respuesta Esperada:</strong> {q.correctAnswer}</p>
                      {q.explanation && <p className="text-slate-600 dark:text-slate-300 mt-1">{q.explanation}</p>}
                    </div>
                  )}
                </div>
              ))}
            </div>

            {/* Finalize Button */}
            {!isExamSubmitted ? (
              <button
                onClick={handleFinalizeExam}
                className="w-full py-3.5 bg-indigo-600 hover:bg-indigo-700 text-white font-bold rounded-xl shadow-md transition-all flex items-center justify-center gap-2"
              >
                <CheckCircle className="w-5 h-5" /> Entregar y Calificar Examen
              </button>
            ) : (
              <button
                onClick={() => setActiveTab('library')}
                className="w-full py-3.5 bg-slate-800 hover:bg-slate-900 text-white font-bold rounded-xl shadow-md transition-all"
              >
                Volver a la Biblioteca
              </button>
            )}
          </div>
        )}

        {/* TAB 4: PRINT / EXPORT MODE */}
        {activeTab === 'print' && activeExam && (
          <div className="p-6 max-w-4xl mx-auto w-full space-y-6">
            <div className="flex items-center justify-between print:hidden">
              <button
                onClick={() => setActiveTab('library')}
                className="text-xs font-semibold text-indigo-600 dark:text-indigo-400 flex items-center gap-1 hover:underline"
              >
                <ArrowLeft className="w-4 h-4" /> Volver
              </button>

              <div className="flex items-center gap-3">
                <label className="flex items-center gap-2 text-xs font-medium text-slate-600 dark:text-slate-300 cursor-pointer">
                  <input
                    type="checkbox"
                    checked={showAnswerKey}
                    onChange={(e) => setShowAnswerKey(e.target.checked)}
                    className="rounded text-indigo-600 focus:ring-indigo-500"
                  />
                  Incluir Hoja de Respuestas (Pauta)
                </label>

                <button
                  onClick={() => window.print()}
                  className="px-4 py-2 bg-indigo-600 text-white rounded-lg text-xs font-bold flex items-center gap-2 shadow-sm hover:bg-indigo-700 transition-all"
                >
                  <Printer className="w-4 h-4" /> Imprimir o Guardar como PDF
                </button>
              </div>
            </div>

            {/* Clean Printable Worksheet View */}
            <div className="bg-white text-slate-900 p-8 rounded-xl shadow-lg border border-slate-200 print:shadow-none print:border-none print:p-0 space-y-6">
              
              {/* Header Box */}
              <div className="border-b-2 border-slate-800 pb-4 flex justify-between items-end">
                <div>
                  <h1 className="text-xl font-bold uppercase tracking-wide">{activeExam.title}</h1>
                  <p className="text-xs text-slate-600 mt-1">Evaluación Académica</p>
                </div>
                <div className="text-right text-xs space-y-1">
                  <p><strong>Alumno/a:</strong> ___________________________</p>
                  <p><strong>Fecha:</strong> _______________ <strong>Nota:</strong> _______</p>
                </div>
              </div>

              {/* Instructions */}
              <p className="text-xs italic text-slate-600">
                Instrucciones: Lee atentamente cada pregunta antes de responder. Responde con claridad.
              </p>

              {/* Questions */}
              <div className="space-y-6 pt-2">
                {activeExam.questions.map((q, idx) => (
                  <div key={q.id || idx} className="space-y-2 text-sm">
                    <p className="font-bold text-slate-900">
                      {idx + 1}. {q.question}
                    </p>

                    {/* Multiple choice options */}
                    {q.options && (
                      <div className="grid grid-cols-2 gap-2 pl-4 pt-1">
                        {q.options.map((opt, oIdx) => (
                          <div key={oIdx} className="flex items-center gap-2 text-xs">
                            <span className="w-4 h-4 border border-slate-400 rounded-full inline-block flex-shrink-0" />
                            <span>{opt}</span>
                          </div>
                        ))}
                      </div>
                    )}

                    {/* Open lines for written response */}
                    {(q.type === 'short_answer' || q.type === 'essay') && (
                      <div className="pt-2 space-y-2">
                        <div className="border-b border-slate-300 h-4" />
                        <div className="border-b border-slate-300 h-4" />
                        {q.type === 'essay' && <div className="border-b border-slate-300 h-4" />}
                      </div>
                    )}
                  </div>
                ))}
              </div>

              {/* Printable Answer Key Sheet if toggled */}
              {showAnswerKey && (
                <div className="pt-8 border-t-2 border-dashed border-slate-400 space-y-4 page-break-before">
                  <h3 className="font-bold text-sm uppercase text-slate-800 bg-slate-100 p-2 rounded">
                    Pauta de Corrección (Exclusivo Profesor/a)
                  </h3>

                  <div className="space-y-3 text-xs">
                    {activeExam.questions.map((q, idx) => (
                      <div key={idx} className="border-b border-slate-100 pb-2">
                        <p className="font-bold">{idx + 1}. {q.question}</p>
                        <p className="text-indigo-700 font-semibold mt-0.5">
                          Respuesta correcta: {q.correctAnswer}
                        </p>
                        {q.explanation && (
                          <p className="text-slate-500 text-[11px] mt-0.5">Explicación: {q.explanation}</p>
                        )}
                      </div>
                    ))}
                  </div>
                </div>
              )}
            </div>
          </div>
        )}

      </main>

      {/* --- MODAL FOR FOLDER CREATION / EDITING --- */}
      {modalOpen && (
        <div className="fixed inset-0 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 z-50">
          <div className={`w-full max-w-md p-6 rounded-2xl shadow-xl border ${darkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-white border-slate-200'}`}>
            <h3 className="text-lg font-bold mb-4">
              {modalMode === 'create' ? 'Crear Nueva Carpeta' : 'Editar Nombre'}
            </h3>

            <form onSubmit={handleSaveFolder} className="space-y-4">
              {modalMode === 'create' && (
                <div>
                  <label className="block text-xs font-semibold text-slate-500 mb-1">Tipo de Nivel:</label>
                  <select
                    value={folderFormType}
                    onChange={(e) => setFolderFormType(e.target.value)}
                    className={`w-full p-2.5 rounded-lg border text-sm ${darkMode ? 'bg-slate-700 border-slate-600' : 'bg-slate-50 border-slate-300'}`}
                  >
                    <option value="course">Curso (ej. 2º Bachillerato)</option>
                    <option value="term">Trimestre / Periodo (ej. Trimestre 1)</option>
                    <option value="topic">Tema (ej. Tema 1: Derivadas)</option>
                  </select>
                </div>
              )}

              <div>
                <label className="block text-xs font-semibold text-slate-500 mb-1">Nombre:</label>
                <input
                  type="text"
                  required
                  placeholder="Ej: Matemáticas II o Trimestre 1"
                  value={folderFormName}
                  onChange={(e) => setFolderFormName(e.target.value)}
                  className={`w-full p-2.5 rounded-lg border text-sm ${darkMode ? 'bg-slate-700 border-slate-600' : 'bg-slate-50 border-slate-300'}`}
                />
              </div>

              <div className="flex justify-end gap-2 pt-2">
                <button
                  type="button"
                  onClick={() => setModalOpen(false)}
                  className="px-4 py-2 rounded-lg text-xs font-semibold hover:bg-slate-100 dark:hover:bg-slate-700 transition-all"
                >
                  Cancelar
                </button>
                <button
                  type="submit"
                  className="px-4 py-2 bg-indigo-600 text-white rounded-lg text-xs font-bold hover:bg-indigo-700 transition-all"
                >
                  Guardar
                </button>
              </div>
            </form>
          </div>
        </div>
      )}

    </div>
  );
}

// --- RECURSIVE FOLDER TREE NODE COMPONENT ---
function FolderNode({ node, selectedTopicId, setSelectedTopicId, onAddSubfolder, onEditFolder, onDeleteFolder, darkMode }) {
  const [isOpen, setIsOpen] = useState(true);

  const isTopic = node.type === 'topic';
  const isSelected = selectedTopicId === node.id;

  return (
    <div className="pl-2 my-0.5">
      <div 
        className={`flex items-center justify-between p-1.5 rounded-lg group text-xs cursor-pointer transition-all ${
          isSelected 
            ? 'bg-indigo-100 text-indigo-700 dark:bg-indigo-950 dark:text-indigo-300 font-bold' 
            : 'hover:bg-slate-100 dark:hover:bg-slate-700/60 text-slate-700 dark:text-slate-300'
        }`}
        onClick={() => {
          if (isTopic) {
            setSelectedTopicId(node.id);
          } else {
            setIsOpen(!isOpen);
          }
        }}
      >
        <div className="flex items-center gap-1.5 min-w-0">
          {!isTopic ? (
            <button 
              type="button"
              onClick={(e) => { e.stopPropagation(); setIsOpen(!isOpen); }}
              className="p-0.5 text-slate-400 hover:text-slate-600"
            >
              {isOpen ? <ChevronDown className="w-3.5 h-3.5" /> : <ChevronRight className="w-3.5 h-3.5" />}
            </button>
          ) : (
            <span className="w-3.5" />
          )}

          {node.type === 'course' && <BookOpen className="w-3.5 h-3.5 text-indigo-500 flex-shrink-0" />}
          {node.type === 'term' && <Layers className="w-3.5 h-3.5 text-amber-500 flex-shrink-0" />}
          {node.type === 'topic' && <Folder className="w-3.5 h-3.5 text-emerald-500 flex-shrink-0" />}

          <span className="truncate">{node.name}</span>
        </div>

        {/* Action icons on hover */}
        <div className="opacity-0 group-hover:opacity-100 flex items-center gap-1">
          {!isTopic && (
            <button
              onClick={(e) => {
                e.stopPropagation();
                const nextType = node.type === 'course' ? 'term' : 'topic';
                onAddSubfolder(node, nextType);
              }}
              title="Añadir Subcarpeta"
              className="p-1 hover:bg-slate-200 dark:hover:bg-slate-600 rounded text-slate-500"
            >
              <Plus className="w-3 h-3" />
            </button>
          )}

          <button
            onClick={(e) => { e.stopPropagation(); onEditFolder(node); }}
            title="Renombrar"
            className="p-1 hover:bg-slate-200 dark:hover:bg-slate-600 rounded text-slate-500"
          >
            <Edit3 className="w-3 h-3" />
          </button>

          <button
            onClick={(e) => { e.stopPropagation(); onDeleteFolder(node.id); }}
            title="Eliminar"
            className="p-1 hover:bg-slate-200 dark:hover:bg-slate-600 rounded text-red-500"
          >
            <Trash2 className="w-3 h-3" />
          </button>
        </div>
      </div>

      {/* Render children recursively if folder is open */}
      {!isTopic && isOpen && node.children && node.children.length > 0 && (
        <div className="border-l border-slate-200 dark:border-slate-700 ml-3">
          {node.children.map(child => (
            <FolderNode
              key={child.id}
              node={child}
              selectedTopicId={selectedTopicId}
              setSelectedTopicId={setSelectedTopicId}
              onAddSubfolder={onAddSubfolder}
              onEditFolder={onEditFolder}
              onDeleteFolder={onDeleteFolder}
              darkMode={darkMode}
            />
          ))}
        </div>
      )}
    </div>
  );
}
