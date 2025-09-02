window.addEventListener('load', () => {

    function isMobileDevice() {

        if (navigator.userAgentData && navigator.userAgentData.mobile) {
            return true;
        }

        const mobileRegex = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i;
        return mobileRegex.test(navigator.userAgent);
    }

    if (!isMobileDevice()) {
        return;
    }

    let tapCount = 0;
    let tapTimer = null;
    const requiredTaps = 5;
    const tapTimeout = 400;

    document.body.addEventListener('touchstart', (e) => {

        if (e.target.closest('#button-zone') || e.target.closest('#joystick-zone')) {
            return;
        }

        tapCount++;

        clearTimeout(tapTimer);

        tapTimer = setTimeout(() => {
            tapCount = 0;
        }, tapTimeout);

        if (tapCount >= requiredTaps) {

            if (typeof showUI === 'function') {
                showUI();
            }

            tapCount = 0;
            clearTimeout(tapTimer);
        }
    });

    const mobileControls = document.getElementById('mobile-controls');
    if (mobileControls) {
        mobileControls.style.display = 'block';
    }

    const buttonZone = document.getElementById('button-zone');
    const joystickZone = document.getElementById('joystick-zone');
    const keyState = {
        up: false,
        down: false,
        left: false,
        right: false
    };
    const keyMap = {
        up: {
            key: 'ArrowUp',
            code: 38
        },
        down: {
            key: 'ArrowDown',
            code: 40
        },
        left: {
            key: 'ArrowLeft',
            code: 37
        },
        right: {
            key: 'ArrowRight',
            code: 39
        }
    };
    let joystick = null;

    function createOrUpdateJoystick() {
        if (joystick) {
            joystick.destroy();
        }
        if (!joystickZone) return;
        const zoneRect = joystickZone.getBoundingClientRect();
        if (zoneRect.width === 0 || zoneRect.height === 0) {
            console.warn('joystickZone has zero size, skipping joystick creation');
            return;
        }

        const joystickSize = Math.max(100, Math.min(zoneRect.width, zoneRect.height) * 0.6);
        const centerLeft = zoneRect.width / 2;
        const centerTop = zoneRect.height / 2;
        joystick = nipplejs.create({
            zone: joystickZone,
            mode: 'static',
            position: {
                left: centerLeft + 'px',
                top: centerTop + 'px'
            },
            size: joystickSize,
            color: 'white',
            threshold: 0.2
        });

        joystick.on('move', (evt, data) => {
            if (!data || !data.vector) return;

            const x = data.vector.x;
            const y = data.vector.y;
            const threshold = 0.3;

            const newKeyState = {
                up: false,
                down: false,
                left: false,
                right: false
            };

            if (y < -threshold) newKeyState.down = true;
            if (y > threshold) newKeyState.up = true;
            if (x < -threshold) newKeyState.left = true;
            if (x > threshold) newKeyState.right = true;

            for (const dir in keyState) {
                if (keyState[dir] && !newKeyState[dir]) {
                    simulateKeyEvent('keyup', keyMap[dir].key, keyMap[dir].code);
                } else if (!keyState[dir] && newKeyState[dir]) {
                    simulateKeyEvent('keydown', keyMap[dir].key, keyMap[dir].code);
                }
                keyState[dir] = newKeyState[dir];
            }
        });

        joystick.on('end', () => {
            for (const dir in keyState) {
                if (keyState[dir]) {
                    simulateKeyEvent('keyup', keyMap[dir].key, keyMap[dir].code);
                    keyState[dir] = false;
                }
            }
        });
    }

    function updateControlPositions() {
        if (!buttonZone || !joystickZone) return;
        const isTabletLayout = (window.innerWidth / window.innerHeight) > 1.3;
        joystickZone.style.cssText = '';
        buttonZone.style.cssText = '';
        if (isTabletLayout) {
            const sideWidth = '25vw';
            joystickZone.style.left = '0';
            joystickZone.style.top = '0';
            joystickZone.style.width = sideWidth;
            joystickZone.style.height = '100%';
            joystickZone.style.background = 'transparent';
            buttonZone.style.right = '0';
            buttonZone.style.top = '0';
            buttonZone.style.width = sideWidth;
            buttonZone.style.height = '100%';
            buttonZone.style.background = 'transparent';
        } else {
            const controlHeight = '30vh';
            joystickZone.style.left = '0';
            joystickZone.style.bottom = '0';
            joystickZone.style.width = '50vw';
            joystickZone.style.height = controlHeight;
            joystickZone.style.background = 'rgba(0,0,0,0.2)';
            buttonZone.style.right = '0';
            buttonZone.style.bottom = '0';
            buttonZone.style.width = '50vw';
            buttonZone.style.height = controlHeight;
            buttonZone.style.background = 'rgba(0,0,0,0.2)';
        }
        createOrUpdateJoystick();
    }
    const resizeObserver = new ResizeObserver(() => {
        setTimeout(() => {
            window.requestAnimationFrame(updateControlPositions);
        }, 100);
    });
    resizeObserver.observe(document.body);
    setTimeout(updateControlPositions, 500);
    const buttons = [{
        id: 'button-z',
        key: 'z',
        keyCode: 90,
        img_up: '/spr/z.svg',
        img_down: '/spr/z_pressed.svg'
    }, {
        id: 'button-x',
        key: 'x',
        keyCode: 88,
        img_up: '/spr/x.svg',
        img_down: '/spr/x_pressed.svg'
    }, {
        id: 'button-c',
        key: 'c',
        keyCode: 67,
        img_up: '/spr/c.svg',
        img_down: '/spr/c_pressed.svg'
    }, ];
    buttons.forEach(buttonInfo => {
        const buttonElement = document.getElementById(buttonInfo.id);
        if (buttonElement) {
            buttonElement.src = buttonInfo.img_up;
            buttonElement.addEventListener('touchstart', (e) => {
                e.preventDefault();
                buttonElement.classList.add('pressed');
                buttonElement.src = buttonInfo.img_down;
                simulateKeyEvent('keydown', buttonInfo.key, buttonInfo.keyCode);
            }, {
                passive: false
            });
            buttonElement.addEventListener('touchend', (e) => {
                e.preventDefault();
                buttonElement.classList.remove('pressed');
                buttonElement.src = buttonInfo.img_up;
                simulateKeyEvent('keyup', buttonInfo.key, buttonInfo.keyCode);
            }, {
                passive: false
            });
        }
    });
});
