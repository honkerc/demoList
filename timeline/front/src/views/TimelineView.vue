<template>
    <div class="app">
        <div class="content">
            <!-- Calendar -->
            <div class="cal-card">
                <div class="cal-header">
                    <button class="cal-nav-btn" @click="calMonth(-1)">←</button>
                    <div class="cal-title">{{ calTitle }}</div>
                    <button class="cal-nav-btn" @click="calMonth(1)">→</button>
                </div>
                <div class="cal-grid">
                    <div class="cal-weekday" v-for="w in weekdays" :key="w">{{ w }}</div>
                    <div v-for="(day, i) in calDays" :key="i" :class="day.classes" @click="goToDate(day.dateStr)">
                        {{ day.label }}
                        <span v-if="day.hasRecord" class="dot"></span>
                    </div>
                </div>
            </div>

            <!-- Summary -->
            <div class="summary-row">
                <div class="summary-item">
                    <div class="value">{{ sumTotal }}</div>
                    <div class="label">总记录</div>
                </div>
                <div class="summary-item">
                    <div class="value">{{ sumCount }}</div>
                    <div class="label">活动数</div>
                </div>
                <div class="summary-item">
                    <div class="value">{{ sumRate }}%</div>
                    <div class="label">利用率</div>
                </div>
            </div>

            <!-- Record Cards -->
            <div class="rc-section">
                <div class="rc-header">记录详情</div>
                <div class="rc-list">
                    <div v-if="todayRecords.length === 0" class="empty-state">
                        <p>暂无记录</p>
                    </div>
                    <div v-for="r in todayRecords" :key="r.id" class="rc-card" @click="deleteRecord(r.id)">
                        <div class="rc-bar" :style="{ background: cc(r.category) }"></div>
                        <div class="rc-body">
                            <div class="rc-top">
                                <span class="rc-name">{{ r.name }}</span>
                                <span class="rc-dur">{{ fmt(r.duration) }}</span>
                            </div>
                            <div class="rc-bottom">
                                <span class="rc-time">{{ r.start || '--:--' }} — {{ r.end || '--:--' }}</span>
                                <span class="rc-cat" :style="{ background: cc(r.category) }">{{ r.category }}</span>
                            </div>
                        </div>
                        <button class="rc-del" @click.stop="deleteRecord(r.id)">✕</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'TimelineView',
    data() {
        return {
            currentDate: new Date(),
            calViewDate: new Date(),
            weekdays: ['日', '一', '二', '三', '四', '五', '六'],
        }
    },
    computed: {
        calTitle() {
            const y = this.calViewDate.getFullYear()
            const m = this.calViewDate.getMonth() + 1
            return y + '年' + m + '月'
        },
        calDays() {
            const records = this.getRecords()
            const recordDates = new Set(records.map(r => r.date))
            const y = this.calViewDate.getFullYear()
            const m = this.calViewDate.getMonth()
            const firstDay = new Date(y, m, 1).getDay()
            const daysInMonth = new Date(y, m + 1, 0).getDate()
            const daysInPrev = new Date(y, m, 0).getDate()
            const today = new Date()
            const todayStr = this.dateStr(today)
            const selStr = this.dateStr(this.currentDate)
            const days = []

            // Previous month days
            for (let i = firstDay - 1; i >= 0; i--) {
                days.push({ label: daysInPrev - i, classes: 'cal-day other', dateStr: '', hasRecord: false })
            }
            // Current month days
            for (let d = 1; d <= daysInMonth; d++) {
                const ds = y + '-' + String(m + 1).padStart(2, '0') + '-' + String(d).padStart(2, '0')
                let cls = 'cal-day'
                if (ds === todayStr) cls += ' today'
                if (ds === selStr) cls += ' active'
                days.push({ label: d, classes: cls, dateStr: ds, hasRecord: recordDates.has(ds) })
            }
            // Next month days
            const totalCells = firstDay + daysInMonth
            const remaining = (7 - (totalCells % 7)) % 7
            for (let d = 1; d <= remaining; d++) {
                days.push({ label: d, classes: 'cal-day other', dateStr: '', hasRecord: false })
            }
            return days
        },
        todayRecords() {
            const records = this.getRecords()
            const ds = this.dateStr(this.currentDate)
            return records.filter(r => r.date === ds).sort((a, b) => (a.start || '00:00').localeCompare(b.start || '00:00'))
        },
        sumTotal() {
            let total = 0
            this.todayRecords.forEach(r => total += r.duration)
            return this.fmt(total)
        },
        sumCount() {
            return this.todayRecords.length
        },
        sumRate() {
            const total = this.todayRecords.reduce((s, r) => s + r.duration, 0)
            const goal = 8 * 3600
            return goal > 0 ? Math.min(100, Math.round(total / goal * 100)) : 0
        }
    },
    methods: {
        dateStr(d) {
            return d.getFullYear() + '-' + String(d.getMonth() + 1).padStart(2, '0') + '-' + String(d.getDate()).padStart(2, '0')
        },
        getRecords() {
            try {
                return JSON.parse(localStorage.getItem('timeline_records') || '[]')
            } catch { return [] }
        },
        saveRecords(r) {
            localStorage.setItem('timeline_records', JSON.stringify(r))
        },
        fmt(s) {
            const h = Math.floor(s / 3600)
            const m = Math.floor((s % 3600) / 60)
            return h > 0 ? h + 'h ' + m + 'm' : m + 'm'
        },
        cc(cat) {
            const m = {
                '工作': 'var(--c1)', '学习': 'var(--c2)', '阅读': 'var(--c3)', '运动': 'var(--c4)',
                '创作': 'var(--c5)', '社交': 'var(--c6)', '休息': 'var(--c7)', '其他': 'var(--c8)',
                '编码': 'var(--c1)', '会议': 'var(--c10)', '文档': 'var(--c16)',
                '课程': 'var(--c2)', '练习': 'var(--c11)', '笔记': 'var(--c13)',
                '书籍': 'var(--c3)', '文章': 'var(--c14)', '论文': 'var(--c13)',
                '跑步': 'var(--c4)', '瑜伽': 'var(--c12)', '骑行': 'var(--c11)',
                '写作': 'var(--c5)', '绘画': 'var(--c9)', '摄影': 'var(--c10)',
                '聚会': 'var(--c6)', '电话': 'var(--c9)', '聊天': 'var(--c14)',
                '午休': 'var(--c7)', '冥想': 'var(--c11)', '散步': 'var(--c13)',
                '通勤': 'var(--c8)', '家务': 'var(--c12)', '杂事': 'var(--c16)'
            }
            return m[cat] || 'var(--text3)'
        },
        calMonth(delta) {
            const d = new Date(this.calViewDate)
            d.setMonth(d.getMonth() + delta)
            this.calViewDate = d
        },
        goToDate(dateStr) {
            if (!dateStr) return
            const parts = dateStr.split('-')
            this.currentDate = new Date(parseInt(parts[0]), parseInt(parts[1]) - 1, parseInt(parts[2]))
        },
        deleteRecord(id) {
            if (confirm('删除这条记录？')) {
                let records = this.getRecords()
                records = records.filter(r => r.id !== id)
                this.saveRecords(records)
            }
        }
    },
    mounted() {
        // Generate mock data for today if none exists
        const records = this.getRecords()
        const todayStr = this.dateStr(new Date())
        const hasToday = records.some(r => r.date === todayStr)
        if (!hasToday) {
            const mockData = [
                { name: '晨间冥想', cat: '休息', dur: 30, startH: 6, startM: 30 },
                { name: '阅读', cat: '阅读', dur: 45, startH: 7, startM: 15 },
                { name: '编码', cat: '工作', dur: 120, startH: 8, startM: 30 },
                { name: '会议', cat: '工作', dur: 45, startH: 10, startM: 45 },
                { name: '课程', cat: '学习', dur: 60, startH: 11, startM: 45 },
                { name: '午休', cat: '休息', dur: 40, startH: 12, startM: 50 },
                { name: '写作', cat: '创作', dur: 90, startH: 14, startM: 0 },
                { name: '跑步', cat: '运动', dur: 35, startH: 15, startM: 45 },
                { name: '笔记', cat: '学习', dur: 50, startH: 16, startM: 30 },
                { name: '散步', cat: '休息', dur: 25, startH: 17, startM: 30 },
                { name: '绘画', cat: '创作', dur: 60, startH: 19, startM: 0 },
                { name: '阅读', cat: '阅读', dur: 40, startH: 20, startM: 15 },
                { name: '冥想', cat: '休息', dur: 20, startH: 21, startM: 30 }
            ]
            const newRecords = []
            mockData.forEach((m, i) => {
                const startMin = m.startH * 60 + m.startM
                const endMin = startMin + m.dur
                const sh = Math.floor(startMin / 60) % 24, sm = startMin % 60
                const eh = Math.floor(endMin / 60) % 24, em = endMin % 60
                newRecords.push({
                    id: Date.now() + i,
                    date: todayStr,
                    start: String(sh).padStart(2, '0') + ':' + String(sm).padStart(2, '0'),
                    end: String(eh).padStart(2, '0') + ':' + String(em).padStart(2, '0'),
                    duration: m.dur * 60,
                    category: m.cat,
                    name: m.name,
                    created_at: todayStr + 'T' + String(sh).padStart(2, '0') + ':' + String(sm).padStart(2, '0') + ':00'
                })
            })
            this.saveRecords(records.concat(newRecords))
        }
    }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap');

:root {
    --bg: #f8f6f3;
    --card: #ffffff;
    --text: #1a1a1e;
    --text2: #6b6b76;
    --text3: #9a9aa6;
    --line: #e8e6e1;
    --soft: #f0eee9;
    --accent: #5b5bd6;
    --c1: #5b5bd6;
    --c2: #3b8fe0;
    --c3: #4cb782;
    --c4: #d4a04a;
    --c5: #e07b5b;
    --c6: #b877c9;
    --c7: #6bb5b5;
    --c8: #9a9aa6;
    --c9: #f06292;
    --c10: #7e57c2;
    --c11: #26c6da;
    --c12: #ff8a65;
    --c13: #aed581;
    --c14: #ffd54f;
    --c15: #ef5350;
    --c16: #78909c;
    --shadow: 0 1px 3px rgba(0, 0, 0, 0.04), 0 1px 2px rgba(0, 0, 0, 0.02);
    --shadow-lg: 0 8px 32px rgba(0, 0, 0, 0.06);
    --radius: 16px;
    --radius-sm: 10px;
}

html.dark {
    --bg: #121214;
    --card: #1e1e22;
    --text: #f0f0f2;
    --text2: #9a9aa6;
    --text3: #6b6b76;
    --line: #2a2a2e;
    --soft: #252529;
    --accent: #7c7ce0;
    --shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
    --shadow-lg: 0 8px 32px rgba(0, 0, 0, 0.3);
}

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

html,
body {
    height: 100%;
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'SF Pro Display', sans-serif;
    background: var(--bg);
    color: var(--text);
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}

.app {
    max-width: 420px;
    margin: 0 auto;
    min-height: 100vh;
    background: var(--bg);
    display: flex;
    flex-direction: column;
    position: relative;
}

.content {
    flex: 1;
    padding: 0 20px 100px;
    overflow-y: auto;
}

.content::-webkit-scrollbar {
    display: none;
}

/* Calendar */
.cal-card {
    background: var(--card);
    border-radius: var(--radius);
    padding: 16px 16px 12px;
    margin-bottom: 16px;
    box-shadow: var(--shadow);
    border: 1px solid var(--line);
}

.cal-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 14px;
}

.cal-header .cal-title {
    font-size: 14px;
    font-weight: 700;
    color: var(--text);
}

.cal-header .cal-nav-btn {
    background: none;
    border: none;
    font-size: 16px;
    color: var(--text3);
    cursor: pointer;
    padding: 4px 8px;
    border-radius: 8px;
    transition: all .2s;
    font-family: inherit;
    font-weight: 600;
}

.cal-header .cal-nav-btn:hover {
    background: var(--soft);
    color: var(--accent);
}

.cal-grid {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 2px;
    text-align: center;
}

.cal-grid .cal-weekday {
    font-size: 10px;
    font-weight: 700;
    color: var(--text3);
    padding: 6px 0;
    letter-spacing: .5px;
}

.cal-grid .cal-day {
    font-size: 12px;
    font-weight: 600;
    color: var(--text2);
    padding: 6px 0;
    border-radius: 8px;
    cursor: pointer;
    transition: all .2s;
    position: relative;
}

.cal-grid .cal-day:hover {
    background: var(--soft);
}

.cal-grid .cal-day.other {
    color: var(--text3);
    opacity: .35;
}

.cal-grid .cal-day.today {
    background: var(--accent);
    color: #fff;
    font-weight: 800;
    box-shadow: 0 2px 8px rgba(91, 91, 214, .25);
}

.cal-grid .cal-day.active {
    background: color-mix(in srgb, var(--accent) 12%, transparent);
    color: var(--accent);
    font-weight: 700;
}

.cal-grid .cal-day .dot {
    width: 4px;
    height: 4px;
    border-radius: 50%;
    background: var(--c3);
    margin: 2px auto 0;
    display: block;
}

.cal-grid .cal-day.today .dot {
    background: rgba(255, 255, 255, .6);
}

/* Summary */
.summary-row {
    display: flex;
    gap: 8px;
    margin-bottom: 20px;
}

.summary-item {
    flex: 1;
    background: var(--card);
    border-radius: var(--radius-sm);
    padding: 14px 12px;
    text-align: center;
    border: 1px solid var(--line);
}

.summary-item .value {
    font-size: 20px;
    font-weight: 800;
    color: var(--text);
}

.summary-item .label {
    font-size: 10px;
    color: var(--text3);
    margin-top: 4px;
    font-weight: 600;
}

/* Record Cards */
.rc-section {
    margin-top: 20px;
}

.rc-header {
    font-size: 12px;
    font-weight: 700;
    color: var(--text3);
    letter-spacing: 1px;
    text-transform: uppercase;
    margin-bottom: 12px;
}

.rc-list {
    display: flex;
    flex-direction: column;
    gap: 8px;
}

.rc-card {
    display: flex;
    align-items: stretch;
    gap: 12px;
    background: var(--card);
    border-radius: var(--radius-sm);
    padding: 14px 16px;
    border: 1px solid var(--line);
    transition: all .2s;
    cursor: pointer;
    position: relative;
}

.rc-card:hover {
    border-color: var(--accent);
}

.rc-card .rc-bar {
    width: 4px;
    border-radius: 4px;
    flex-shrink: 0;
}

.rc-card .rc-body {
    flex: 1;
    min-width: 0;
}

.rc-card .rc-top {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
}

.rc-card .rc-name {
    font-size: 15px;
    font-weight: 700;
    color: var(--text);
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

.rc-card .rc-dur {
    font-size: 12px;
    font-weight: 700;
    color: var(--text2);
    font-variant-numeric: tabular-nums;
    flex-shrink: 0;
}

.rc-card .rc-bottom {
    display: flex;
    align-items: center;
    gap: 8px;
    margin-top: 4px;
}

.rc-card .rc-time {
    font-size: 11px;
    font-weight: 600;
    color: var(--text3);
    font-variant-numeric: tabular-nums;
}

.rc-card .rc-cat {
    font-size: 10px;
    font-weight: 700;
    padding: 2px 10px;
    border-radius: 999px;
    color: #fff;
}

.rc-card .rc-del {
    position: absolute;
    top: 8px;
    right: 8px;
    width: 22px;
    height: 22px;
    border-radius: 50%;
    border: none;
    background: var(--soft);
    color: var(--text3);
    font-size: 12px;
    cursor: pointer;
    display: none;
    align-items: center;
    justify-content: center;
    font-family: inherit;
    transition: all .2s;
}

.rc-card:hover .rc-del {
    display: flex;
}

.rc-card .rc-del:hover {
    background: var(--accent2);
    color: #fff;
}

.empty-state {
    text-align: center;
    padding: 40px 20px;
}

.empty-state p {
    font-size: 14px;
    color: var(--text3);
    font-weight: 500;
}

@media(min-width:768px) {
    .app {
        margin: 20px auto;
        min-height: 90vh;
        overflow: hidden;
        box-shadow: var(--shadow-lg);
    }
}
</style>
