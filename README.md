# pages/header.py
# coding: utf-8
"""
HeaderWidget  ─ System / Status / Running-Rate / Log
====================================================
PySide6 + QFluentWidgets 기반 헤더 모듈
"""

from PySide6.QtCore import Qt, QSize
from PySide6.QtGui import QColor, QPainter, QPen
from PySide6.QtWidgets import (
    QWidget, QLabel, QVBoxLayout, QHBoxLayout,
    QGridLayout, QTextEdit, QProgressBar
)
from qfluentwidgets import (
    PrimaryPushButton, TogglePushButton, FluentIcon
)


# ──────────────────────────────────────────────
# 🔹 작은 동그란 LED 위젯  (빨/초/주/회)
# ──────────────────────────────────────────────
class StatusLED(QWidget):
    COLOR = {
        "red":    "#d9534f",
        "green":  "#5cb85c",
        "orange": "#f0ad4e",
        "grey":   "#9e9e9e"
    }

    def __init__(self, color="red", parent=None):
        super().__init__(parent)
        self._color = color
        self.setFixedSize(14, 14)

    def setColor(self, color: str):
        if color in self.COLOR:
            self._color = color
            self.update()

    # paint circular LED
    def paintEvent(self, e):
        painter = QPainter(self)
        painter.setRenderHint(QPainter.Antialiasing)
        painter.setPen(QPen(Qt.transparent))
        painter.setBrush(QColor(self.COLOR[self._color]))
        r = min(self.width(), self.height()) / 2 - 1
        painter.drawEllipse(self.rect().center(), r, r)


# ──────────────────────────────────────────────
# 🔹 Part-1 : SYSTEM
# ──────────────────────────────────────────────
class SystemPane(QWidget):
    """Server / Scheduler / Long-Run 상태"""

    def __init__(self, parent=None):
        super().__init__(parent)
        lay = QVBoxLayout(self, spacing=6)
        lay.addWidget(self._title("SYSTEM"))

        grid = QGridLayout()
        grid.setHorizontalSpacing(6)
        grid.setVerticalSpacing(4)

        # ─ server connect
        self.serverLed = StatusLED("red")
        self.serverBtn = PrimaryPushButton("Connect")
        grid.addWidget(self.serverLed, 0, 0, Qt.AlignLeft)
        grid.addWidget(QLabel("Server"), 0, 1, Qt.AlignLeft)
        grid.addWidget(self.serverBtn, 0, 2)

        # ─ scheduler
        self.schedLed = StatusLED("red")
        self.schedToggle = TogglePushButton("Scheduler")
        grid.addWidget(self.schedLed, 1, 0, Qt.AlignLeft)
        grid.addWidget(QLabel("Scheduler"), 1, 1, Qt.AlignLeft)
        grid.addWidget(self.schedToggle, 1, 2)

        # ─ long-run status
        self.longRunLed = StatusLED("red")
        grid.addWidget(self.longRunLed, 2, 0, Qt.AlignLeft)
        grid.addWidget(QLabel("Long Run"), 2, 1, Qt.AlignLeft)

        lay.addLayout(grid)
        lay.addStretch()

        # demo toggle
        self.serverBtn.clicked.connect(self._toggleServer)
        self.schedToggle.toggled.connect(
            lambda s: self.schedLed.setColor("green" if s else "red")
        )

    @staticmethod
    def _title(text: str):
        lbl = QLabel(text)
        lbl.setStyleSheet("font-weight:bold; font-size:14px;")
        return lbl

    def _toggleServer(self):
        if self.serverBtn.text() == "Connect":
            self.serverBtn.setText("Disconnect")
            self.serverLed.setColor("green")
        else:
            self.serverBtn.setText("Connect")
            self.serverLed.setColor("red")


# ──────────────────────────────────────────────
# 🔹 Part-2 : STATUS  (Error / Manual / Auto)
# ──────────────────────────────────────────────
class StatusPane(QWidget):
    def __init__(self, parent=None):
        super().__init__(parent)
        lay = QVBoxLayout(self, spacing=6)
        lay.addWidget(self._title("STATUS"))

        grid = QGridLayout()
        grid.setHorizontalSpacing(6)

        self.errLed, self.errCnt = self._addRow(grid, 0, "Error",   "red")
        self.manLed, self.manCnt = self._addRow(grid, 1, "Manual",  "orange")
        self.autLed, self.autCnt = self._addRow(grid, 2, "Auto",    "green")

        lay.addLayout(grid)
        lay.addStretch()

    @staticmethod
    def _title(text: str):
        lbl = QLabel(text)
        lbl.setStyleSheet("font-weight:bold; font-size:14px;")
        return lbl

    def _addRow(self, grid: QGridLayout, row: int, name: str, color: str):
        led = StatusLED(color)
        cnt = QLabel("0")
        cnt.setStyleSheet("font-size:13px;")
        grid.addWidget(led, row, 0, Qt.AlignLeft)
        grid.addWidget(QLabel(name), row, 1, Qt.AlignLeft)
        grid.addWidget(cnt, row, 2, Qt.AlignLeft)
        return led, cnt


# ──────────────────────────────────────────────
# 🔹 Part-3 : RUNNING RATE (Progress %)
# ──────────────────────────────────────────────
class RunningRatePane(QWidget):
    def __init__(self, parent=None):
        super().__init__(parent)
        lay = QVBoxLayout(self, spacing=6)
        lay.addWidget(self._title("RUNNING RATE"))

        self.bar = QProgressBar(self)
        self.bar.setRange(0, 100)
        self.bar.setValue(0)
        self.bar.setAlignment(Qt.AlignCenter)
        self.bar.setFixedHeight(18)
        lay.addWidget(self.bar)
        lay.addStretch()

    @staticmethod
    def _title(text: str):
        lbl = QLabel(text)
        lbl.setStyleSheet("font-weight:bold; font-size:14px;")
        return lbl


# ──────────────────────────────────────────────
# 🔹 Part-4 : LOG BOX (가로폭 3)
# ──────────────────────────────────────────────
class LogPane(QWidget):
    def __init__(self, parent=None):
        super().__init__(parent)
        lay = QVBoxLayout(self, spacing=4)
        lay.addWidget(self._title("LOG"))

        self.box = QTextEdit(self)
        self.box.setReadOnly(True)
        self.box.setPlaceholderText("Logs will appear here…")
        self.box.setStyleSheet(
            "background:#1e1e1e; color:#e6e6e6; font-family:Consolas,monospace;"
        )
        lay.addWidget(self.box, 1)

    @staticmethod
    def _title(text: str):
        lbl = QLabel(text)
        lbl.setStyleSheet("font-weight:bold; font-size:14px;")
        return lbl

    # 외부에서 로그 추가용
    def add(self, text: str):
        self.box.append(text)


# ──────────────────────────────────────────────
# 🔹 HeaderWidget : 네 파트 수평 배치
# ──────────────────────────────────────────────
class HeaderWidget(QWidget):
    """System / Status / Running-Rate / Log   1:1:1:3"""

    def __init__(self, parent=None):
        super().__init__(parent)
        self.setObjectName("HeaderWidget")

        hLay = QHBoxLayout(self)
        hLay.setContentsMargins(10, 10, 10, 10)
        hLay.setSpacing(8)

        self.system   = SystemPane(self)
        self.status   = StatusPane(self)
        self.running  = RunningRatePane(self)
        self.logPane  = LogPane(self)

        hLay.addWidget(self.system,  1)
        hLay.addWidget(self.status,  1)
        hLay.addWidget(self.running, 1)
        hLay.addWidget(self.logPane, 3)

        # 간단 스타일
        self.setStyleSheet("""
            QWidget#HeaderWidget{ background:#202020; }
            QLabel{ color:#ffffff; }
        """)