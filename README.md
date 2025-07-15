<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>تطبيق تفاعلي: قانون يوم القرآن الكريم</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Calm Harmony (Warm neutrals, deep greens, muted gold accents) -->
    <!-- Application Structure Plan: A single-page application designed for intuitive exploration. It uses a top navigation bar for quick access to thematic sections: a hero section for the introduction, a section for visualizing the law's objectives with an interactive chart, a grid for displaying planned activities, a simple diagram for the implementation mechanism, and a collapsible accordion for the full legal text. This structure breaks down the dense legal text into digestible, user-friendly modules, facilitating better understanding compared to a linear document. The interactive 3D Quran browsing experience is restored as the primary hero section, with "Allah" text on the cover. A new prominent, animated overlay displays a key Quranic verse for spiritual impact, separate from the 3D book pages. -->
    <!-- Visualization & Content Choices: 
        - Interactive 3D Quran Browsing -> Engage/Inform -> Three.js Canvas -> Mouse drag for camera, buttons for page turning -> Provides a unique, immersive entry point, symbolizing the Quran's centrality. Quran cover is shiny gold with "Allah" text. Pages are plain.
        - Animated Quranic Verse Overlay -> Engage/Inform -> HTML/CSS/JS -> Words fade/slide in with a delay -> Delivers a powerful spiritual message directly to the user with an engaging visual effect, separate from the 3D book.
        - Preamble -> Inform -> Hero Section Text -> N/A -> Clearly sets the tone and purpose upfront.
        - Article 3 (Objectives) -> Organize/Compare -> Interactive Bar Chart & Cards (Chart.js) -> Clicking a bar highlights the corresponding objective's card -> Transforms a simple list into an engaging, interactive visualization, making the goals memorable.
        - Article 4 (Activities) -> Organize/Inform -> Icon-based Grid (HTML/Tailwind) -> Hover effects -> Provides a quick, scannable overview of the diverse events, more appealing than a bulleted list.
        - Article 5 (Implementation) -> Organize -> Flow Diagram (HTML/Tailwind) -> N/A -> Simplifies the complex relationships between implementing bodies into an easy-to-read visual flow.
        - Full Law Text -> Inform/Reference -> Accordion (HTML/JS) -> Clicking expands articles -> Offers access to the detailed legal text without cluttering the main interface, catering to users needing full details.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Tajawal', sans-serif;
            scroll-behavior: smooth;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 600px;
            margin-left: auto;
            margin-right: auto;
            height: 400px;
            max-height: 50vh;
        }
        .section-title::after {
            content: '';
            display: block;
            width: 80px;
            height: 3px;
            background-color: #c79935;
            margin: 1rem auto 0;
            border-radius: 2px;
        }
        .accordion-content {
            max-height: 0;
            overflow: hidden;
            transition: max-height 0.5s ease-in-out;
        }
        #quran-3d-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: #e0f2f7;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
        }
        #quran-3d-canvas {
            display: block;
            width: 100%;
            height: 100%;
        }
        #loading-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(255, 255, 255, 0.9);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 100;
            flex-direction: column;
            color: #107a57;
            font-weight: bold;
        }
        .spinner {
            border: 4px solid rgba(16, 122, 87, 0.1);
            border-top: 4px solid #107a57;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        /* Styles for the animated verse overlay */
        #verse-overlay {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 20; /* Above 3D scene, below buttons/text */
            text-align: center;
            color: #FFD700; /* Golden color for the verse */
            font-size: 2.5rem; /* Larger font */
            font-weight: bold;
            line-height: 1.6;
            max-width: 80%;
            opacity: 0; /* Initially hidden */
            transition: opacity 1s ease-in-out;
            text-shadow: 0 0 10px rgba(255, 215, 0, 0.5), 0 0 20px rgba(255, 215, 0, 0.3); /* Subtle glow */
        }

        #verse-overlay span {
            display: inline-block;
            opacity: 0;
            transform: translateY(20px);
            animation: fadeInSlideUp 0.8s forwards;
            animation-fill-mode: backwards; /* Keep element at start state before animation */
        }

        @keyframes fadeInSlideUp {
            from {
                opacity: 0;
                transform: translateY(20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
    </style>
</head>
<body class="bg-stone-50 text-gray-800">

    <nav id="navbar" class="bg-white/80 backdrop-blur-md shadow-sm sticky top-0 z-50">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex items-center justify-between h-16">
                <div class="flex-shrink-0">
                    <h1 class="text-xl font-bold text-emerald-800">يوم القرآن الكريم</h1>
                </div>
                <div class="hidden md:block">
                    <div class="ml-10 flex items-baseline space-x-4 space-x-reverse">
                        <a href="#hero-3d" class="text-gray-600 hover:bg-emerald-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium">الواجهة التفاعلية</a>
                        <a href="#introduction" class="text-gray-600 hover:bg-emerald-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium">مقدمة القانون</a>
                        <a href="#objectives" class="text-gray-600 hover:bg-emerald-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium">الأهداف</a>
                        <a href="#activities" class="text-gray-600 hover:bg-emerald-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium">الفعاليات</a>
                        <a href="#implementation" class="text-gray-600 hover:bg-emerald-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium">التنفيذ</a>
                        <a href="#full-text" class="text-gray-600 hover:bg-emerald-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium">النص الكامل</a>
                    </div>
                </div>
            </div>
        </div>
    </nav>

    <header id="hero-3d" class="relative w-full h-[80vh] max-h-[800px] flex flex-col justify-center items-center text-center overflow-hidden">
        <div id="quran-3d-container" class="absolute inset-0">
            <div id="loading-overlay">
                <div class="spinner"></div>
                <p class="mt-4">جاري تحميل الواجهة التفاعلية...</p>
            </div>
        </div>
        
        <div id="verse-overlay" class="font-tajawal">
            <!-- Verse will be dynamically inserted here -->
        </div>

        <div class="relative z-10 text-white p-8 rounded-lg bg-black/50 backdrop-blur-sm">
            <h2 class="text-4xl lg:text-6xl font-bold leading-tight">يوم القرآن الكريم</h2>
            <p class="mt-4 text-lg lg:text-xl max-w-3xl mx-auto">
                استكشف قدسية القرآن الكريم في تجربة تفاعلية فريدة.
            </p>
            <div class="mt-8 flex justify-center space-x-4 space-x-reverse">
                <button id="prevPage" class="bg-emerald-700 text-white font-bold py-2 px-6 rounded-lg shadow-lg hover:bg-emerald-800 transition-colors duration-300">
                    الصفحة السابقة
                </button>
                <button id="nextPage" class="bg-emerald-700 text-white font-bold py-2 px-6 rounded-lg shadow-lg hover:bg-emerald-800 transition-colors duration-300">
                    الصفحة التالية
                </button>
            </div>
            <div class="mt-4">
                <a href="#introduction" class="inline-block bg-amber-600 text-white font-bold py-3 px-8 rounded-lg shadow-lg hover:bg-amber-700 transition-colors duration-300">
                    اكتشف القانون
                </a>
            </div>
        </div>
    </header>

    <section id="introduction" class="bg-emerald-50 py-20 lg:py-32">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 text-center">
            <h2 class="text-4xl lg:text-6xl font-bold text-emerald-900 leading-tight">مقدمة قانون يوم القرآن الكريم</h2>
            <p class="mt-6 max-w-3xl mx-auto text-lg text-gray-700">
                إن الغاية من هذا القانون هي تكريم القرآن الكريم، كتاب الله الخالد، وتعزيز قيم التسامح والرحمة والعدل التي يدعو إليها، وتذكير الأجيال بأهمية تعاليمه في بناء مجتمع فاضل ومتماسك.
            </p>
        </div>
    </section>

    <main>
        <section id="objectives" class="py-20 bg-white">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="text-center">
                    <h2 class="text-3xl font-bold text-emerald-900 section-title">أهداف القانون</h2>
                    <p class="mt-8 max-w-2xl mx-auto text-lg text-gray-600">
                        يهدف هذا القانون إلى تحقيق مجموعة من الغايات النبيلة التي تعزز مكانة القرآن الكريم في المجتمع. يستعرض هذا القسم الأهداف الخمسة الرئيسية التي يسعى القانون لتحقيقها، موضحاً الرؤية الكامنة وراء كل هدف.
                    </p>
                </div>

                <div class="mt-16 grid grid-cols-1 lg:grid-cols-2 gap-12 items-center">
                    <div class="chart-container">
                        <canvas id="objectivesChart"></canvas>
                    </div>
                    <div id="objectives-cards" class="space-y-4">
                    </div>
                </div>
            </div>
        </section>

        <section id="activities" class="py-20 bg-stone-50">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="text-center">
                    <h2 class="text-3xl font-bold text-emerald-900 section-title">الفعاليات والأنشطة المقترحة</h2>
                     <p class="mt-8 max-w-2xl mx-auto text-lg text-gray-600">
                        لتحقيق أهداف القانون، تُقام مجموعة متنوعة من الفعاليات والأنشطة في يوم القرآن الكريم. هذا القسم يعرض أبرز هذه الأنشطة التي تهدف لإشراك كافة فئات المجتمع.
                    </p>
                </div>

                <div id="activities-grid" class="mt-16 grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
                </div>
            </div>
        </section>
        
        <section id="implementation" class="py-20 bg-white">
             <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="text-center">
                    <h2 class="text-3xl font-bold text-emerald-900 section-title">آلية التنفيذ والمسؤوليات</h2>
                    <p class="mt-8 max-w-2xl mx-auto text-lg text-gray-600">
                        نجاح تطبيق هذا القانون يعتمد على آلية تنفيذ واضحة وتنسيق فعال بين الجهات المعنية. يوضح هذا المخطط الهيكل التنظيمي والمسؤوليات المنصوص عليها في القانون.
                    </p>
                </div>

                <div class="mt-16 flex flex-col items-center">
                    <div class="bg-emerald-700 text-white p-6 rounded-xl shadow-lg text-center w-full max-w-md">
                        <h3 class="text-xl font-bold">ديوان الوقف الشيعي وديوان الوقف السني</h3>
                        <p class="text-sm">(الجهات الرئيسية المسؤولة عن التنفيذ)</p>
                    </div>
                    
                    <div class="h-12 w-1 bg-emerald-300 my-4"></div>
                    
                    <div class="text-center text-lg font-semibold text-emerald-800 mb-4">بالتنسيق مع</div>

                    <div class="w-full grid grid-cols-1 md:grid-cols-3 gap-8 text-center">
                        <div class="bg-emerald-50 p-6 rounded-xl shadow-md border-t-4 border-emerald-500">
                            <h4 class="font-bold text-emerald-800">وزارة التربية</h4>
                        </div>
                        <div class="bg-emerald-50 p-6 rounded-xl shadow-md border-t-4 border-emerald-500">
                            <h4 class="font-bold text-emerald-800">وزارة التعليم العالي والبحث العلمي</h4>
                        </div>
                         <div class="bg-emerald-50 p-6 rounded-xl shadow-md border-t-4 border-emerald-500">
                            <h4 class="font-bold text-emerald-800">وزارة الثقافة والسياحة والآثار</h4>
                        </div>
                        <!-- The section for Diwan Al-Waqf Al-Shia will now only contain its main title -->
                        <div class="bg-emerald-50 p-6 rounded-xl shadow-md border-t-4 border-emerald-500 md:col-span-3 lg:col-span-1">
                            <h4 class="font-bold text-emerald-800">ديوان الوقف الشيعي</h4>
                        </div>
                    </div>
                </div>
             </div>
        </section>

        <section id="full-text" class="py-20 bg-stone-50">
            <div class="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="text-center">
                    <h2 class="text-3xl font-bold text-emerald-900 section-title">النص الكامل لمسودة القانون</h2>
                    <p class="mt-8 max-w-2xl mx-auto text-lg text-gray-600">
                       للمهتمين بالتفاصيل القانونية الكاملة، يوفر هذا القسم النص الكامل لجميع مواد مسودة القانون. يمكنك تصفح المواد والاطلاع على تفاصيلها.
                    </p>
                </div>
                
                <div id="accordion-container" class="mt-16 space-y-4">
                </div>
            </div>
        </section>
    </main>

    <footer class="bg-emerald-900 text-white py-8">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 text-center">
            <p>تطبيق تفاعلي لعرض مسودة قانون يوم القرآن الكريم.</p>
            <p class="text-sm text-emerald-300 mt-2">&copy; 2025 - تم إنشاؤه لأغراض العرض التوضيحي.
                <br>رئيس مهندسين: رائد ابراهيم خليل ابراهيم
                <br>دائرة أوقاف المحافظات
                <br>قسم المحافظات الجنوبية
            </p>
        </div>
    </footer>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            // Three.js Quran Scene Setup
            const quran3dContainer = document.getElementById('quran-3d-container');
            const loadingOverlay = document.getElementById('loading-overlay');
            const verseOverlay = document.getElementById('verse-overlay');
            let scene, camera, renderer, quranBook, leftPageMesh, rightPageMesh;
            let mouseX = 0, mouseY = 0;
            let targetX = 0, targetY = 0;
            const windowHalfX = window.innerWidth / 2;
            const windowHalfY = window.innerHeight / 2;
            let currentPageIndex = 0;
            const totalSimulatedPages = 20; 

            const quranicVerse = `شَهْرُ رَمَضَانَ الَّذِي أُنزِلَ فِيهِ الْقُرْآنُ هُدًى لِّلنَّاسِ وَبَيِّنَاتٍ مِّنَ الْهُدَىٰ وَالْفُرْقَانِ <br>(البقرة: 185)`;

            const pageContents = [];
            // Add other simulated pages (starting from 1, as the verse is separate)
            for (let i = 0; i < totalSimulatedPages; i++) {
                pageContents.push(`
                    بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيمِ
                    <br><br>
                    هذه صفحة رقم ${i + 1} من القرآن الكريم.
                    <br><br>
                    لَا يَأْتِيهِ الْبَاطِلُ مِن بَيْنِ يَدَيْهِ وَلَا مِنْ خَلْفِهِ ۖ تَنزِيلٌ مِّنْ حَكِيمٍ حَمِيدٍ.
                    <br><br>
                    (توضيح: هذا النص هو مثال توضيحي لمحاكاة شكل الآيات، وليس آيات قرآنية حقيقية بالترتيب.)
                `);
            }
            const totalPagesInBook = pageContents.length;

            // Function to create a CanvasTexture for Quran pages (now plain)
            function createPageTexture(text = "", width = 512, height = 1024) {
                const canvas = document.createElement('canvas');
                canvas.width = width;
                canvas.height = height;
                const context = canvas.getContext('2d');

                context.fillStyle = '#fdfbf2'; // Page background color (off-white)
                context.fillRect(0, 0, width, height);

                // Optional: add page number or simple line art if desired
                if (text) {
                    context.fillStyle = '#888'; // Grey for page numbers
                    context.font = '20px Tajawal';
                    context.textAlign = 'center';
                    context.textBaseline = 'bottom';
                    context.fillText(text, width / 2, height - 20);
                }

                return new THREE.CanvasTexture(canvas);
            }

            // Function to create a CanvasTexture for the Quran cover with "الله"
            function createCoverTexture(text = "الله", width = 512, height = 1024) {
                const canvas = document.createElement('canvas');
                canvas.width = width;
                canvas.height = height;
                const context = canvas.getContext('2d');

                context.fillStyle = '#DAA520'; // Solid gold background for the cover
                context.fillRect(0, 0, width, height);

                context.fillStyle = '#FFFFFF'; // White text for contrast on gold
                context.font = 'bold 120px Tajawal';
                context.textAlign = 'center';
                context.textBaseline = 'middle';
                context.fillText(text, width / 2, height / 2);

                return new THREE.CanvasTexture(canvas);
            }


            function initThreeJS() {
                scene = new THREE.Scene();
                scene.background = new THREE.Color(0xe0f2f7);

                camera = new THREE.PerspectiveCamera(75, quran3dContainer.clientWidth / quran3dContainer.clientHeight, 0.1, 1000);
                camera.position.set(0, 0, 3);

                renderer = new THREE.WebGLRenderer({ antialias: true });
                renderer.setSize(quran3dContainer.clientWidth, quran3dContainer.clientHeight);
                renderer.setPixelRatio(window.devicePixelRatio);
                quran3dContainer.appendChild(renderer.domElement);
                renderer.domElement.id = 'quran-3d-canvas';

                const ambientLight = new THREE.AmbientLight(0xffffff, 0.7);
                scene.add(ambientLight);

                const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
                directionalLight.position.set(5, 10, 7);
                scene.add(directionalLight);

                createQuranBook();
                updateQuranDisplay(); // Initial display of pages (now plain)

                quran3dContainer.addEventListener('mousemove', onDocumentMouseMove);
                window.addEventListener('resize', onWindowResize);

                animate();
                loadingOverlay.style.display = 'none';
                displayAnimatedVerse(quranicVerse); // Display the animated verse after Three.js loads
            }

            function createQuranBook() {
                quranBook = new THREE.Group();
                const bookThickness = 0.2;
                const pageWidth = 1.5;
                const pageHeight = 2;

                // Book cover (front) with "الله" text
                const frontCoverTexture = createCoverTexture("الله");
                const frontCoverMaterial = new THREE.MeshStandardMaterial({
                    map: frontCoverTexture,
                    roughness: 0.2,
                    metalness: 0.8,
                    clearcoat: 1,
                    clearcoatRoughness: 0.1
                });
                const coverGeometry = new THREE.BoxGeometry(pageWidth, pageHeight, bookThickness / 2);
                const frontCover = new THREE.Mesh(coverGeometry, frontCoverMaterial);
                frontCover.position.z = bookThickness / 4;
                quranBook.add(frontCover);

                // Book cover (back) - solid gold
                const backCoverMaterial = new THREE.MeshStandardMaterial({
                    color: 0xFFD700, // Gold color
                    roughness: 0.2,
                    metalness: 0.8,
                    clearcoat: 1,
                    clearcoatRoughness: 0.1
                });
                const backCover = new THREE.Mesh(coverGeometry, backCoverMaterial);
                backCover.position.z = -bookThickness / 4;
                quranBook.add(backCover);

                // Left Page Mesh (now plain)
                const pageGeometry = new THREE.PlaneGeometry(pageWidth * 0.95, pageHeight * 0.95);
                leftPageMesh = new THREE.Mesh(pageGeometry, new THREE.MeshBasicMaterial({ map: createPageTexture(`صفحة ${currentPageIndex + 2}`) }));
                leftPageMesh.position.set(-pageWidth * 0.475, 0, 0.01);
                quranBook.add(leftPageMesh);

                // Right Page Mesh (now plain)
                rightPageMesh = new THREE.Mesh(pageGeometry, new THREE.MeshBasicMaterial({ map: createPageTexture(`صفحة ${currentPageIndex + 1}`) }));
                rightPageMesh.position.set(pageWidth * 0.475, 0, 0.01);
                quranBook.add(rightPageMesh);
                
                scene.add(quranBook);

                quranBook.rotation.y = Math.PI / 8;
                quranBook.rotation.x = -Math.PI / 16;
            }

            function onDocumentMouseMove(event) {
                // Calculate targetX and targetY based on mouse position
                targetX = (event.clientX - windowHalfX) / 200;
                targetY = (event.clientY - windowHalfY) / 200;
            }

            function onWindowResize() {
                camera.aspect = quran3dContainer.clientWidth / quran3dContainer.clientHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(quran3dContainer.clientWidth, quran3dContainer.clientHeight);
            }

            function animate() {
                requestAnimationFrame(animate);

                // Smoothly interpolate current rotation towards target rotation
                // Increased damping factor to make it settle faster and more rigidly
                quranBook.rotation.y += (targetX - quranBook.rotation.y) * 0.1; // Damping factor 0.1
                quranBook.rotation.x += (targetY - quranBook.rotation.x) * 0.1; // Damping factor 0.1
                
                renderer.render(scene, camera);
            }

            const prevPageBtn = document.getElementById('prevPage');
            const nextPageBtn = document.getElementById('nextPage');

            prevPageBtn.addEventListener('click', () => {
                if (currentPageIndex > 0) {
                    currentPageIndex -= 2;
                    updateQuranDisplay();
                }
            });

            nextPageBtn.addEventListener('click', () => {
                if (currentPageIndex + 2 < totalPagesInBook) {
                    currentPageIndex += 2;
                    updateQuranDisplay();
                } else if (currentPageIndex + 1 < totalPagesInBook) {
                    currentPageIndex += 1;
                    updateQuranDisplay();
                }
            });

            function updateQuranDisplay() {
                if (leftPageMesh && rightPageMesh) {
                    // Dispose previous textures to prevent memory leaks
                    if (rightPageMesh.material.map) rightPageMesh.material.map.dispose();
                    if (leftPageMesh.material.map) leftPageMesh.material.map.dispose();

                    // Right page content (simulated page number)
                    rightPageMesh.material.map = createPageTexture(`صفحة ${currentPageIndex + 1}`);
                    rightPageMesh.material.needsUpdate = true;
                    rightPageMesh.visible = true;

                    // Left page content (simulated page number)
                    if (currentPageIndex + 1 < totalPagesInBook) {
                        leftPageMesh.material.map = createPageTexture(`صفحة ${currentPageIndex + 2}`);
                        leftPageMesh.material.needsUpdate = true;
                        leftPageMesh.visible = true;
                    } else {
                        // If no content for the left page, show a blank page
                        leftPageMesh.material.map = createPageTexture("");
                        leftPageMesh.material.needsUpdate = true;
                        leftPageMesh.visible = true;
                    }
                }
            }
            
            // Function to display the animated verse
            function displayAnimatedVerse(verse) {
                verseOverlay.innerHTML = ''; // Clear previous content
                const words = verse.split(' ');
                words.forEach((word, index) => {
                    const span = document.createElement('span');
                    span.textContent = word + ' ';
                    span.style.animationDelay = `${index * 0.1}s`; // Staggered animation
                    verseOverlay.appendChild(span);
                });
                verseOverlay.style.opacity = 1; // Make the overlay visible
            }

            window.onload = function() {
                initThreeJS();
            };

            const lawData = {
                objectives: [
                    { title: "تكريم القرآن الكريم", description: "إبراز مكانة القرآن الكريم ككتاب هداية وتشريع لجميع البشر." },
                    { title: "تعزيز القيم الإسلامية", description: "نشر قيم التسامح، والرحمة، والعدل، والإحسان، والسلام، والتكافل الاجتماعي المستوحاة من القرآن." },
                    { title: "التوعية بأهمية القرآن", description: "تشجيع جميع أفراد المجتمع على قراءة القرآن الكريم وتدبر آياته والعمل بها." },
                    { title: "نشر ثقافة الاعتدال", description: "محاربة الغلو والتطرف، وترسيخ مفاهيم الوسطية والاعتدال التي يدعو إليها القرآن الكريم." },
                    { title: "دعم الأنشطة القرآنية", description: "تشجيع ودعم المؤسسات والهيئات التي تعمل على خدمة القرآن الكريم وتعليمه." },
                ],
                activities: [
                    { name: "المسابقات القرآنية", icon: "🏆" },
                    { name: "الندوات والمؤتمرات", icon: "🏛️" },
                    { name: "المحاضرات والخطب", icon: "🎤" },
                    { name: "المعارض القرآنية", icon: "🖼️" },
                    { name: "برامج التلاوة الجماعية", icon: "📖" },
                    { name: "التكريم والاحتفاء", icon: "🏅" },
                    { name: "الأنشطة الإعلامية", icon: "📡" },
                    { name: "دعم المؤسسات", icon: "🤝" }
                ],
                articles: [
                    { title: "المادة 1: التسمية والتعريف", content: "يسمى هذا القانون 'قانون يوم القرآن الكريم'. ويُقصد بيوم القرآن الكريم اليوم المحدد للاحتفال بالقرآن الكريم وتكريم منزلته." },
                    { title: "المادة 2: تحديد يوم القرآن الكريم", content: "يُحدد يوم 17 من شهر رمضان المبارك من كل عام هجري يومًا وطنيًا للقرآن الكريم في جمهورية العراق." },
                    { title: "المادة 3: أهداف يوم القرآن الكريم", content: "يهدف الاحتفال بيوم القرآن الكريم إلى تحقيق الأهداف التالية: (1) تكريم القرآن الكريم (2) تعزيز القيم الإسلامية (3) التوعية بأهمية القرآن (4) نشر ثقافة الاعتدال (5) دعم الأنشطة القرآنية." },
                    { title: "المادة 4: الفعاليات والأنشطة", content: "تُقام في يوم القرآن الكريم مجموعة من الفعاليات والأنشطة في جميع أنحاء جمهورية العراق، وتشمل على سبيل المثال لا الحصر: المسابقات، الندوات، المحاضرات، المعارض، برامج التلاوة، التكريم، والأنشطة الإعلامية." },
                    { title: "المادة 5: مسؤولية التنفيذ", content: "1. تتولى ديوان الوقف الشيعي وديوان الوقف السني بالتنسيق مع وزارة التربية ووزارة التعليم العالي والبحث العلمي ووزارة الثقافة والسياحة والآثار، ومؤسسات المجتمع المدني ذات الصلة، وضع الخطط والبرامج اللازمة للاحتفال بيوم القرآن الكريم والإشراف على تنفيذها. 2. يُخصص جزء من ميزانيات الوزارات والمؤسسات المعنية لدعم وتمويل الأنشطة والفعاليات الخاصة بيوم القرآن الكريم." },
                    { title: "المادة 6: العقوبات", content: "تُحدد عقوبات مناسبة لمن يتعمد الإساءة للقرآن الكريم أو انتهاك حرمته، وفقًا للقوانين النافذة في جمهورية العراق." },
                    { title: "المادة 7: أحكام عامة وختامية", content: "1. يُعمل بهذا القانون من تاريخ نشره في الجريدة الرسمية. 2. يُلغى كل نص يتعارض مع أحكام هذا القانون. 3. يجوز إصدار تعليمات لتسهيل تنفيذ أحكام هذا القانون." },
                ]
            };
            
            const objectivesCardsContainer = document.getElementById('objectives-cards');
            lawData.objectives.forEach((obj, index) => {
                const card = document.createElement('div');
                card.className = 'objective-card bg-white p-6 rounded-lg shadow-md border-l-4 border-emerald-500 transition-all duration-300';
                card.id = `objective-card-${index}`;
                card.innerHTML = `<h3 class="font-bold text-xl text-emerald-800">${obj.title}</h3><p class="mt-2 text-gray-600">${obj.description}</p>`;
                objectivesCardsContainer.appendChild(card);
            });

            const ctx = document.getElementById('objectivesChart').getContext('2d');
            const objectivesChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: lawData.objectives.map(obj => obj.title),
                    datasets: [{
                        label: 'أهمية الهدف',
                        data: [1, 1, 1, 1, 1],
                        backgroundColor: 'rgba(16, 122, 87, 0.6)',
                        borderColor: 'rgba(11, 89, 64, 1)',
                        borderWidth: 1,
                        hoverBackgroundColor: 'rgba(199, 153, 53, 0.8)',
                        hoverBorderColor: 'rgba(199, 153, 53, 1)',
                    }]
                },
                options: {
                    indexAxis: 'y',
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: {
                            display: false,
                        },
                        y: {
                           ticks: {
                                font: {
                                    family: "'Tajawal', sans-serif",
                                    size: 14,
                                }
                           }
                        }
                    },
                    plugins: {
                        legend: {
                            display: false
                        },
                        tooltip: {
                            enabled: false
                        }
                    },
                    onHover: (event, chartElement) => {
                        const canvas = event.native.target;
                        canvas.style.cursor = chartElement[0] ? 'pointer' : 'default';
                        
                        document.querySelectorAll('.objective-card').forEach(card => {
                            card.classList.remove('bg-amber-50', 'shadow-xl', 'scale-105');
                            card.classList.add('bg-white');
                        });

                        if (chartElement.length > 0) {
                            const activeIndex = chartElement[0].index;
                            const activeCard = document.getElementById(`objective-card-${activeIndex}`);
                            if(activeCard) {
                                activeCard.classList.add('bg-amber-50', 'shadow-xl', 'scale-105');
                                activeCard.classList.remove('bg-white');
                            }
                        }
                    }
                }
            });

            const activitiesGrid = document.getElementById('activities-grid');
            lawData.activities.forEach(activity => {
                const item = document.createElement('div');
                item.className = 'bg-white p-6 rounded-xl shadow-md text-center hover:shadow-xl hover:-translate-y-1 transition-all duration-300 cursor-pointer';
                item.innerHTML = `
                    <div class="text-5xl">${activity.icon}</div>
                    <h3 class="mt-4 font-semibold text-emerald-800">${activity.name}</h3>
                `;
                activitiesGrid.appendChild(item);
            });

            const accordionContainer = document.getElementById('accordion-container');
            lawData.articles.forEach((article, index) => {
                const item = document.createElement('div');
                item.className = 'border border-gray-200 rounded-lg overflow-hidden bg-white';
                item.innerHTML = `
                    <button class="accordion-header w-full text-right p-5 flex justify-between items-center bg-gray-50 hover:bg-gray-100 focus:outline-none">
                        <span class="font-semibold text-lg text-emerald-800">${article.title}</span>
                        <span class="accordion-icon text-emerald-700 font-bold text-2xl transition-transform duration-300">+</span>
                    </button>
                    <div class="accordion-content">
                        <p class="p-5 text-gray-700 bg-white">${article.content}</p>
                    </div>
                `;
                accordionContainer.appendChild(item);
            });
            
            document.querySelectorAll('.accordion-header').forEach(button => {
                button.addEventListener('click', () => {
                    const content = button.nextElementSibling;
                    const icon = button.querySelector('.accordion-icon');
                    
                    if (content.style.maxHeight) {
                        content.style.maxHeight = null;
                        icon.style.transform = 'rotate(0deg)';
                    } else {
                        document.querySelectorAll('.accordion-content').forEach(c => c.style.maxHeight = null);
                        document.querySelectorAll('.accordion-icon').forEach(i => i.style.transform = 'rotate(0deg)');
                        content.style.maxHeight = content.scrollHeight + "px";
                        icon.style.transform = 'rotate(45deg)';
                    }
                });
            });

        });
    </script>
</body>
</html>
```
لقد قمت بتعديل الكود لنقل "دائرة أوقاف المحافظات" و "قسم المحافظات الجنوبية" من قسم "آلية التنفيذ والمسؤوليات" إلى قسم التذييل (الـ footer)، ليظهروا بعد اسم المهندس.

الآن، ستجد هذه المعلومات مجمعة في نهاية الصف# The-Holly-Quraan-Day.
