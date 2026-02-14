
// 激光布局游戏性设计原则：
// 1. 始终保证有可通关路径
// 2. 形成"观察-等待-冲刺"的节奏
// 3. 渐进难度：从宽通道到窄通道，从慢速到快速
// 4. 每关有独特的挑战主题

function generateLaserLayout(lvl, isScrolling, worldW) {
    const lasers = [];
    const baseInt = Math.max(600, 2000 - lvl * 12);
    
    // 安全区域：起点和终点附近不能放激光
    const safeZoneStart = 80;
    const safeZoneEnd = isScrolling ? worldW - 80 : W - 80;
    
    // 激光类型根据关卡选择
    const maxType = Math.min(4, Math.floor((lvl - 1) / 15));
    
    if (!isScrolling) {
        // ========== 普通关卡布局 ==========
        // 将屏幕分成3-5个横向通道
        const numRows = Math.min(5, 3 + Math.floor(lvl / 20));
        const rowHeight = (H - 100) / numRows;
        
        for (let row = 0; row < numRows; row++) {
            const y = 50 + row * rowHeight + rowHeight / 2;
            const laserType = Math.min(maxType, Math.floor(row / 2));
            
            // 每行生成1-2个激光段，中间留出通道
            const gapSize = Math.max(60, 120 - lvl * 0.5); // 关卡越高，通道越窄
            const numSegments = Math.random() > 0.5 ? 2 : 1;
            
            if (numSegments === 1) {
                // 单个激光段，留出单边通道
                const isLeftGap = Math.random() > 0.5;
                const laserWidth = W - gapSize - (isLeftGap ? 0 : gapSize);
                const laserX = isLeftGap ? gapSize : 0;
                
                // 确保不在安全区
                if (laserX + laserWidth > safeZoneStart && laserX < safeZoneEnd) {
                    lasers.push(new Laser(laserX, y, laserWidth, 5, laserType, baseInt, row * 200));
                }
            } else {
                // 两个激光段，中间留通道
                const sideWidth = (W - gapSize) / 2;
                lasers.push(new Laser(0, y, sideWidth, 5, laserType, baseInt, row * 200));
                lasers.push(new Laser(W - sideWidth, y, sideWidth, 5, laserType, baseInt, row * 200 + 100));
            }
        }
        
        // 高难度关卡添加纵向激光（作为额外挑战）
        if (lvl > 15) {
            const numCols = Math.min(3, Math.floor((lvl - 15) / 10));
            for (let col = 0; col < numCols; col++) {
                const x = 150 + col * ((W - 300) / (numCols + 1));
                const gapY = 100 + Math.random() * (H - 200);
                const gapSize = 50;
                
                // 上半段
                lasers.push(new Laser(x, 0, 5, gapY - gapSize/2, maxType, baseInt * 1.2, col * 150));
                // 下半段
                lasers.push(new Laser(x, gapY + gapSize/2, 5, H - gapY - gapSize/2, maxType, baseInt * 1.2, col * 150 + 300));
            }
        }
    } else {
        // ========== 卷轴关卡布局 ==========
        const numGroups = 3 + Math.floor((lvl - 20) / 15);
        
        for (let g = 0; g < numGroups; g++) {
            const groupX = 250 + g * (worldW - 500) / numGroups;
            const groupType = (lvl + g) % 4;
            
            if (groupType === 0) {
                // 类型0：横向双激光，中间留通道
                const gapSize = 70;
                lasers.push(new Laser(groupX - 150, H/2 - gapSize/2, 300, 5, maxType, baseInt, g * 250));
                lasers.push(new Laser(groupX - 150, H/2 + gapSize/2, 300, 5, maxType, baseInt, g * 250 + baseInt/2));
            } else if (groupType === 1) {
                // 类型1：三横激光，形成上下两个通道
                const gapY1 = H/3;
                const gapY2 = H * 2/3;
                lasers.push(new Laser(groupX - 120, gapY1 - 10, 240, 5, maxType, baseInt, g * 300));
                lasers.push(new Laser(groupX - 120, gapY1 + 10, 240, 5, maxType, baseInt, g * 300 + baseInt/3));
                lasers.push(new Laser(groupX - 120, gapY2 + 10, 240, 5, maxType, baseInt, g * 300 + baseInt*2/3));
            } else if (groupType === 2) {
                // 类型2：纵向激光阵列
                for (let i = 0; i < 3; i++) {
                    const x = groupX - 60 + i * 60;
                    const gapY = 80 + Math.random() * (H - 160);
                    const gapSize = 50;
                    lasers.push(new Laser(x, 0, 5, gapY - gapSize/2, maxType, baseInt * 1.3, g * 200 + i * 100));
                    lasers.push(new Laser(x, gapY + gapSize/2, 5, H - gapY - gapSize/2, maxType, baseInt * 1.3, g * 200 + i * 100));
                }
            } else {
                // 类型3：斜向通道（之字形）
                const segmentWidth = 40;
                for (let i = 0; i < 4; i++) {
                    const y = 60 + i * ((H - 120) / 4);
                    const offset = (i % 2 === 0) ? -20 : 20;
                    lasers.push(new Laser(groupX - 100 + offset, y, 200, 5, maxType, baseInt, g * 250 + i * 150));
                }
            }
        }
    }
    
    return lasers;
}
