<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D地球自转与公转模型演示</title>
    <script src="https://cdn.jsdelivr.net/npm/three@0.132.2/build/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.132.2/examples/js/controls/OrbitControls.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        body {
            background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
            color: #fff;
            min-height: 100vh;
            padding: 20px;
            overflow-x: hidden;
        }
        
        .container {
            display: flex;
            max-width: 1600px;
            margin: 0 auto;
            gap: 20px;
        }
        
        .control-panel {
            flex: 1;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 15px;
            padding: 20px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            max-height: 90vh;
            overflow-y: auto;
        }
        
        .animation-area {
            flex: 2;
            background: rgba(0, 0, 0, 0.3);
            border-radius: 15px;
            padding: 0;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
            position: relative;
            overflow: hidden;
            height: 80vh;
        }
        
        #earthCanvas {
            width: 100%;
            height: 100%;
            display: block;
        }
        
        h1 {
            text-align: center;
            margin-bottom: 30px;
            font-size: 28px;
            text-shadow: 0 2px 10px rgba(0, 0, 0, 0.5);
        }
        
        h2 {
            margin: 20px 0 15px;
            padding-bottom: 10px;
            border-bottom: 2px solid rgba(255, 255, 255, 0.2);
            font-size: 22px;
        }
        
        .control-group {
            margin-bottom: 25px;
        }
        
        .slider-container {
            display: flex;
            flex-direction: column;
            margin-bottom: 15px;
        }
        
        .slider-label {
            display: flex;
            justify-content: space-between;
            margin-bottom: 8px;
        }
        
        .slider-label span {
            font-size: 14px;
        }
        
        input[type="range"] {
            width: 100%;
            height: 8px;
            border-radius: 5px;
            background: rgba(255, 255, 255, 0.1);
            outline: none;
            -webkit-appearance: none;
        }
        
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 18px;
            height: 18px;
            border-radius: 50%;
            background: #4facfe;
            cursor: pointer;
            box-shadow: 0 0 10px rgba(79, 172, 254, 0.8);
        }
        
        .checkbox-group {
            display: flex;
            flex-wrap: wrap;
            gap: 15px;
            margin-top: 10px;
        }
        
        .checkbox-item {
            display: flex;
            align-items: center;
        }
        
        .checkbox-item input {
            margin-right: 8px;
        }
        
        .info-box {
            background: rgba(0, 0, 0, 0.3);
            border-radius: 10px;
            padding: 15px;
            margin-top: 20px;
            font-size: 14px;
            line-height: 1.6;
        }
        
        .info-box h3 {
            margin-bottom: 10px;
            color: #4facfe;
        }
        
        .legend {
            position: absolute;
            bottom: 30px;
            right: 30px;
            background: rgba(0, 0, 0, 0.6);
            padding: 15px;
            border-radius: 10px;
            font-size: 14px;
            z-index: 100;
        }
        
        .legend-item {
            display: flex;
            align-items: center;
            margin-bottom: 8px;
        }
        
        .legend-color {
            width: 15px;
            height: 15px;
            border-radius: 50%;
            margin-right: 10px;
        }
        
        .btn-group {
            display: flex;
            gap: 10px;
            margin-top: 20px;
        }
        
        button {
            flex: 1;
            padding: 12px;
            border: none;
            border-radius: 8px;
            background: linear-gradient(to right, #4facfe, #00f2fe);
            color: white;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
        }
        
        button:hover {
            transform: translateY(-3px);
            box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
        }
        
        button:active {
            transform: translateY(1px);
        }
        
        .reset-btn {
            background: linear-gradient(to right, #ff758c, #ff7eb3);
        }
        
        .stats {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(0, 0, 0, 0.6);
            padding: 15px;
            border-radius: 10px;
            font-size: 14px;
            z-index: 100;
        }
        
        @media (max-width: 1100px) {
            .container {
                flex-direction: column;
            }
            
            .animation-area {
                min-height: 500px;
            }
            
            .control-panel {
                max-height: none;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="control-panel">
            <h1>3D地球自转与公转模型</h1>
            
            <div class="control-group">
                <h2>动画控制</h2>
                <div class="slider-container">
                    <div class="slider-label">
                        <span>动画速度</span>
                        <span id="speedValue">1.0x</span>
                    </div>
                    <input type="range" id="speedControl" min="0.1" max="3" step="0.1" value="1">
                </div>
                
                <div class="btn-group">
                    <button id="playPauseBtn">暂停</button>
                    <button class="reset-btn" id="resetBtn">重置</button>
                </div>
            </div>
            
            <div class="control-group">
                <h2>显示选项</h2>
                <div class="checkbox-group">
                    <div class="checkbox-item">
                        <input type="checkbox" id="showOrbit" checked>
                        <label for="showOrbit">显示公转轨道</label>
                    </div>
                    <div class="checkbox-item">
                        <input type="checkbox" id="showAxis" checked>
                        <label for="showAxis">显示自转轴</label>
                    </div>
                    <div class="checkbox-item">
                        <input type="checkbox" id="showGrid" checked>
                        <label for="showGrid">显示经纬网格</label>
                    </div>
                    <div class="checkbox-item">
                        <input type="checkbox" id="showAtmosphere" checked>
                        <label for="showAtmosphere">显示大气层</label>
                    </div>
                    <div class="checkbox-item">
                        <input type="checkbox" id="showStars" checked>
                        <label for="showStars">显示星空背景</label>
                    </div>
                    <div class="checkbox-item">
                        <input type="checkbox" id="showInfo" checked>
                        <label for="showInfo">显示信息标签</label>
                    </div>
                </div>
            </div>
            
            <div class="control-group">
                <h2>参数设置</h2>
                <div class="slider-container">
                    <div class="slider-label">
                        <span>黄赤交角 (23.5°)</span>
                        <span id="angleValue">23.5°</span>
                    </div>
                    <input type="range" id="angleControl" min="20" max="30" step="0.5" value="23.5">
                </div>
                
                <div class="slider-container">
                    <div class="slider-label">
                        <span>地球大小</span>
                        <span id="sizeValue">100%</span>
                    </div>
                    <input type="range" id="sizeControl" min="50" max="150" step="5" value="100">
                </div>
                
                <div class="slider-container">
                    <div class="slider-label">
                        <span>轨道半径</span>
                        <span id="orbitSizeValue">100%</span>
                    </div>
                    <input type="range" id="orbitSizeControl" min="70" max="130" step="5" value="100">
                </div>
                
                <div class="slider-container">
                    <div class="slider-label">
                        <span>视角距离</span>
                        <span id="cameraValue">100%</span>
                    </div>
                    <input type="range" id="cameraControl" min="50" max="150" step="5" value="100">
                </div>
            </div>
            
            <div class="info-box">
                <h3>地球运动参数</h3>
                <p>• 自转周期: 23小时56分4秒 (恒星日)</p>
                <p>• 公转周期: 365天6小时9分10秒 (恒星年)</p>
                <p>• 黄赤交角: 23°26' (实际值)</p>
                <p>• 公转轨道偏心率: 0.0167</p>
                <p>• 平均公转速度: 29.78 km/s</p>
                <p>• 自转轴倾斜: 导致季节变化</p>
            </div>
        </div>
        
        <div class="animation-area">
            <div id="earthCanvas"></div>
            <div class="stats" id="statsPanel">
                <div>月份: <span id="currentMonth">1月</span></div>
                <div>季节: <span id="currentSeason">北半球: 冬季 | 南半球: 夏季</span></div>
                <div>自转角度: <span id="rotationAngle">0°</span></div>
                <div>公转角度: <span id="revolutionAngle">0°</span></div>
                <div>地轴倾角: <span id="axisAngle">23.5°</span></div>
            </div>
            
            <div class="legend">
                <div class="legend-item">
                    <div class="legend-color" style="background: #4facfe;"></div>
                    <span>地球</span>
                </div>
                <div class="legend-item">
                    <div class="legend-color" style="background: #ff9a9e;"></div>
                    <span>太阳</span>
                </div>
                <div class="legend-item">
                    <div class="legend-color" style="background: #00f2fe;"></div>
                    <span>自转轴</span>
                </div>
                <div class="legend-item">
                    <div class="legend-color" style="background: #ffd700;"></div>
                    <span>公转轨道</span>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 获取控制元素
        const speedControl = document.getElementById('speedControl');
        const speedValue = document.getElementById('speedValue');
        const playPauseBtn = document.getElementById('playPauseBtn');
        const resetBtn = document.getElementById('resetBtn');
        const showOrbit = document.getElementById('showOrbit');
        const showAxis = document.getElementById('showAxis');
        const showGrid = document.getElementById('showGrid');
        const showAtmosphere = document.getElementById('showAtmosphere');
        const showStars = document.getElementById('showStars');
        const showInfo = document.getElementById('showInfo');
        const angleControl = document.getElementById('angleControl');
        const angleValue = document.getElementById('angleValue');
        const sizeControl = document.getElementById('sizeControl');
        const sizeValue = document.getElementById('sizeValue');
        const orbitSizeControl = document.getElementById('orbitSizeControl');
        const orbitSizeValue = document.getElementById('orbitSizeValue');
        const cameraControl = document.getElementById('cameraControl');
        const cameraValue = document.getElementById('cameraValue');
        
        // 统计信息元素
        const currentMonth = document.getElementById('currentMonth');
        const currentSeason = document.getElementById('currentSeason');
        const rotationAngle = document.getElementById('rotationAngle');
        const revolutionAngle = document.getElementById('revolutionAngle');
        const axisAngle = document.getElementById('axisAngle');
        
        // 动画状态
        let isPlaying = true;
        let animationSpeed = 1.0;
        let earthRotation = 0;
        let earthRevolution = 0;
        
        // 地球参数
        let earthSize = 1.0;
        let orbitRadius = 10.0;
        let axialTilt = 23.5;
        let cameraDistance = 1.0;
        
        // Three.js变量
        let scene, camera, renderer, controls;
        let earth, sun, earthAxis, orbit, atmosphere, stars;
        let earthGroup, earthSphere, axisGroup;
        
        // 创建星空背景
        function createStars() {
            const starsGeometry = new THREE.BufferGeometry();
            const starsCount = 5000;
            const positions = new Float32Array(starsCount * 3);
            
            for (let i = 0; i < starsCount * 3; i++) {
                positions[i] = (Math.random() - 0.5) * 2000;
            }
            
            starsGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
            
            const starsMaterial = new THREE.PointsMaterial({
                color: 0xFFFFFF,
                size: 1.5,
                sizeAttenuation: true
            });
            
            stars = new THREE.Points(starsGeometry, starsMaterial);
            scene.add(stars);
        }
        
        // 创建地轴
        function createEarthAxis() {
            const axisGroup = new THREE.Group();
            
            // 地轴主体 - 垂直方向
            const axisGeometry = new THREE.CylinderGeometry(0.02, 0.02, 2.5, 8);
            const axisMaterial = new THREE.MeshBasicMaterial({ color: 0x00f2fe });
            const axis = new THREE.Mesh(axisGeometry, axisMaterial);
            axisGroup.add(axis);
            
            // 北极点标记
            const northPoleGeometry = new THREE.SphereGeometry(0.05, 8, 8);
            const northPoleMaterial = new THREE.MeshBasicMaterial({ color: 0x00f2fe });
            const northPole = new THREE.Mesh(northPoleGeometry, northPoleMaterial);
            northPole.position.set(0, 1.25, 0);
            axisGroup.add(northPole);
            
            // 南极点标记
            const southPoleGeometry = new THREE.SphereGeometry(0.05, 8, 8);
            const southPoleMaterial = new THREE.MeshBasicMaterial({ color: 0x00f2fe });
            const southPole = new THREE.Mesh(southPoleGeometry, southPoleMaterial);
            southPole.position.set(0, -1.25, 0);
            axisGroup.add(southPole);
            
            return axisGroup;
        }
        
        // 创建文字精灵
        function createTextSprite(text, color = '#00f2fe') {
            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');
            canvas.width = 256;
            canvas.height = 128;
            
            context.fillStyle = 'rgba(0, 0, 0, 0)';
            context.fillRect(0, 0, canvas.width, canvas.height);
            
            context.font = 'Bold 60px Arial';
            context.fillStyle = color;
            context.textAlign = 'center';
            context.textBaseline = 'middle';
            context.fillText(text, canvas.width / 2, canvas.height / 2);
            
            const texture = new THREE.CanvasTexture(canvas);
            const material = new THREE.SpriteMaterial({ 
                map: texture,
                transparent: true
            });
            
            return new THREE.Sprite(material);
        }
        
        // 更新地轴倾斜
        function updateAxisTilt() {
            // 地轴应该相对于轨道平面倾斜，但方向保持不变
            // 我们通过旋转整个地球组来实现这一点
            earthGroup.rotation.x = axialTilt * Math.PI / 180;
        }
        
        // 初始化Three.js场景
        function initThreeJS() {
            // 创建场景
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x000011);
            
            // 创建相机
            camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 15, 25);
            
            // 创建渲染器
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(document.getElementById('earthCanvas').offsetWidth, document.getElementById('earthCanvas').offsetHeight);
            renderer.shadowMap.enabled = true;
            document.getElementById('earthCanvas').appendChild(renderer.domElement);
            
            // 添加轨道控制器
            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;
            controls.dampingFactor = 0.05;
            
            // 创建星空
            createStars();
            
            // 添加光源
            const ambientLight = new THREE.AmbientLight(0x333333, 0.3);
            scene.add(ambientLight);
            
            const sunLight = new THREE.PointLight(0xffffff, 2, 100);
            sunLight.castShadow = true;
            scene.add(sunLight);
            
            // 创建太阳
            const sunGeometry = new THREE.SphereGeometry(3, 32, 32);
            const sunMaterial = new THREE.MeshBasicMaterial({ 
                color: 0xff4500,
                emissive: 0xff4500,
                emissiveIntensity: 1.0
            });
            sun = new THREE.Mesh(sunGeometry, sunMaterial);
            sun.castShadow = true;
            scene.add(sun);
            
            // 添加太阳光晕效果
            const sunGlowGeometry = new THREE.SphereGeometry(3.2, 32, 32);
            const sunGlowMaterial = new THREE.MeshBasicMaterial({
                color: 0xff8800,
                transparent: true,
                opacity: 0.3
            });
            const sunGlow = new THREE.Mesh(sunGlowGeometry, sunGlowMaterial);
            sun.add(sunGlow);
            
            // 添加太阳标签
            const sunLabel = createTextSprite("太阳", "#ff4500");
            sunLabel.position.set(0, 4.5, 0);
            sunLabel.scale.set(1, 0.5, 1);
            sun.add(sunLabel);
            
            // 创建地球组 - 用于公转和地轴倾斜
            earthGroup = new THREE.Group();
            scene.add(earthGroup);
            
            // 创建地球球体组 - 用于自转
            earthSphere = new THREE.Group();
            earthGroup.add(earthSphere);
            
            // 创建地球
            const earthGeometry = new THREE.SphereGeometry(1, 32, 32);
            const earthTexture = new THREE.TextureLoader().load('https://threejs.org/examples/textures/planets/earth_atmos_2048.jpg');
            const earthMaterial = new THREE.MeshPhongMaterial({ 
                map: earthTexture,
                specular: new THREE.Color(0x333333),
                shininess: 5
            });
            earth = new THREE.Mesh(earthGeometry, earthMaterial);
            earth.castShadow = true;
            earth.receiveShadow = true;
            earthSphere.add(earth);
            
            // 设置地球初始位置（在轨道上）
            earthGroup.position.x = orbitRadius;
            
            // 创建大气层
            const atmosphereGeometry = new THREE.SphereGeometry(1.02, 32, 32);
            const atmosphereMaterial = new THREE.MeshPhongMaterial({ 
                color: 0x88ccff,
                transparent: true,
                opacity: 0.2,
                side: THREE.BackSide
            });
            atmosphere = new THREE.Mesh(atmosphereGeometry, atmosphereMaterial);
            earthSphere.add(atmosphere);
            
            // 创建地轴
            earthAxis = createEarthAxis();
            earthSphere.add(earthAxis);
            
            // 添加地球标签
            const earthLabel = createTextSprite("地球", "#4facfe");
            earthLabel.position.set(0, 1.8, 0);
            earthLabel.scale.set(0.8, 0.4, 1);
            earthSphere.add(earthLabel);
            
            // 创建经纬网格
            const gridGeometry = new THREE.SphereGeometry(1.01, 12, 8);
            const gridMaterial = new THREE.MeshBasicMaterial({ 
                color: 0xffffff,
                wireframe: true,
                transparent: true,
                opacity: 0.3
            });
            const earthGrid = new THREE.Mesh(gridGeometry, gridMaterial);
            earthSphere.add(earthGrid);
            
            // 创建公转轨道
            const orbitGeometry = new THREE.RingGeometry(orbitRadius - 0.1, orbitRadius + 0.1, 64);
            const orbitMaterial = new THREE.MeshBasicMaterial({ 
                color: 0xffd700,
                transparent: true,
                opacity: 0.3,
                side: THREE.DoubleSide
            });
            orbit = new THREE.Mesh(orbitGeometry, orbitMaterial);
            orbit.rotation.x = -Math.PI / 2;
            scene.add(orbit);
            
            // 初始设置地轴倾斜
            updateAxisTilt();
            
            // 窗口大小调整事件
            window.addEventListener('resize', onWindowResize);
        }
        
        // 窗口大小调整
        function onWindowResize() {
            camera.aspect = document.getElementById('earthCanvas').offsetWidth / document.getElementById('earthCanvas').offsetHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(document.getElementById('earthCanvas').offsetWidth, document.getElementById('earthCanvas').offsetHeight);
        }
        
        // 动画循环
        function animate() {
            requestAnimationFrame(animate);
            
            if (isPlaying) {
                // 更新地球自转 - 地球球体围绕Y轴旋转
                earthSphere.rotation.y += 0.01 * animationSpeed;
                earthRotation += 0.01 * animationSpeed;
                
                // 更新地球公转 - 地球组在轨道上移动
                earthRevolution += 0.002 * animationSpeed;
                earthGroup.position.x = Math.cos(earthRevolution) * orbitRadius;
                earthGroup.position.z = Math.sin(earthRevolution) * orbitRadius;
                
                // 更新统计信息
                updateStats();
            }
            
            // 更新控制器
            controls.update();
            
            // 渲染场景
            renderer.render(scene, camera);
        }
        
        // 更新统计信息
        function updateStats() {
            // 更新角度显示
            rotationAngle.textContent = `${(earthRotation * 180 / Math.PI % 360).toFixed(1)}°`;
            revolutionAngle.textContent = `${(earthRevolution * 180 / Math.PI % 360).toFixed(1)}°`;
            axisAngle.textContent = `${axialTilt}°`;
            
            // 更新月份和季节
            const monthNames = ['1月', '2月', '3月', '4月', '5月', '6月', '7月', '8月', '9月', '10月', '11月', '12月'];
            const monthIndex = Math.floor((earthRevolution / (Math.PI * 2)) * 12) % 12;
            currentMonth.textContent = monthNames[monthIndex];
            
            let season = '';
            if (monthIndex >= 2 && monthIndex <= 4) season = '北半球: 春季 | 南半球: 秋季';
            else if (monthIndex >= 5 && monthIndex <= 7) season = '北半球: 夏季 | 南半球: 冬季';
            else if (monthIndex >= 8 && monthIndex <= 10) season = '北半球: 秋季 | 南半球: 春季';
            else season = '北半球: 冬季 | 南半球: 夏季';
            
            currentSeason.textContent = season;
        }
        
        // 更新显示值
        speedControl.addEventListener('input', function() {
            animationSpeed = parseFloat(this.value);
            speedValue.textContent = animationSpeed.toFixed(1) + 'x';
        });
        
        angleControl.addEventListener('input', function() {
            axialTilt = parseFloat(this.value);
            angleValue.textContent = axialTilt.toFixed(1) + '°';
            // 更新地轴倾斜
            updateAxisTilt();
        });
        
        sizeControl.addEventListener('input', function() {
            earthSize = parseInt(this.value) / 100;
            sizeValue.textContent = parseInt(this.value) + '%';
            earth.scale.set(earthSize, earthSize, earthSize);
            atmosphere.scale.set(earthSize, earthSize, earthSize);
        });
        
        orbitSizeControl.addEventListener('input', function() {
            const orbitSizePercent = parseInt(this.value);
            orbitSizeValue.textContent = orbitSizePercent + '%';
            orbitRadius = 10 * (orbitSizePercent / 100);
            
            // 更新轨道位置
            const currentAngle = Math.atan2(earthGroup.position.z, earthGroup.position.x);
            earthGroup.position.x = Math.cos(currentAngle) * orbitRadius;
            earthGroup.position.z = Math.sin(currentAngle) * orbitRadius;
            
            // 更新轨道可视化
            scene.remove(orbit);
            const orbitGeometry = new THREE.RingGeometry(orbitRadius - 0.1, orbitRadius + 0.1, 64);
            const orbitMaterial = new THREE.MeshBasicMaterial({ 
                color: 0xffd700,
                transparent: true,
                opacity: 0.3,
                side: THREE.DoubleSide
            });
            orbit = new THREE.Mesh(orbitGeometry, orbitMaterial);
            orbit.rotation.x = -Math.PI / 2;
            scene.add(orbit);
        });
        
        cameraControl.addEventListener('input', function() {
            cameraDistance = parseInt(this.value) / 100;
            cameraValue.textContent = parseInt(this.value) + '%';
            camera.position.set(0, 15 * cameraDistance, 25 * cameraDistance);
        });
        
        // 显示/隐藏控制
        showOrbit.addEventListener('change', function() {
            orbit.visible = this.checked;
        });
        
        showAxis.addEventListener('change', function() {
            earthAxis.visible = this.checked;
        });
        
        showGrid.addEventListener('change', function() {
            // 经纬网格是地球球体的第三个子元素（索引2）
            earthSphere.children.forEach((child, index) => {
                if (child.isMesh && child.material.wireframe) {
                    child.visible = this.checked;
                }
            });
        });
        
        showAtmosphere.addEventListener('change', function() {
            atmosphere.visible = this.checked;
        });
        
        showStars.addEventListener('change', function() {
            stars.visible = this.checked;
        });
        
        showInfo.addEventListener('change', function() {
            document.getElementById('statsPanel').style.display = this.checked ? 'block' : 'none';
        });
        
        // 播放/暂停按钮
        playPauseBtn.addEventListener('click', function() {
            isPlaying = !isPlaying;
            this.textContent = isPlaying ? '暂停' : '播放';
        });
        
        // 重置按钮
        resetBtn.addEventListener('click', function() {
            earthRotation = 0;
            earthRevolution = 0;
            earthSphere.rotation.y = 0;
            earthGroup.position.x = orbitRadius;
            earthGroup.position.z = 0;
            
            speedControl.value = 1.0;
            animationSpeed = 1.0;
            speedValue.textContent = '1.0x';
            
            angleControl.value = 23.5;
            axialTilt = 23.5;
            angleValue.textContent = '23.5°';
            updateAxisTilt();
            
            sizeControl.value = 100;
            earthSize = 1.0;
            sizeValue.textContent = '100%';
            earth.scale.set(earthSize, earthSize, earthSize);
            
            orbitSizeControl.value = 100;
            orbitRadius = 10.0;
            orbitSizeValue.textContent = '100%';
            earthGroup.position.x = orbitRadius;
            
            cameraControl.value = 100;
            cameraDistance = 1.0;
            cameraValue.textContent = '100%';
            camera.position.set(0, 15, 25);
            
            controls.reset();
            
            if (!isPlaying) {
                isPlaying = true;
                playPauseBtn.textContent = '暂停';
            }
        });
        
        // 初始化并启动动画
        initThreeJS();
        animate();
    </script>
</body>
</html>
