import sys
import shutil
import os
from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QPushButton, QFileDialog, QLabel, QProgressBar, QListWidget, QComboBox
from PyQt5.QtCore import Qt
from PyQt5.QtGui import QDropEvent

class FileCopyApp(QWidget):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        layout = QVBoxLayout()

        # Добавляем выбор языка
        self.language_selector = QComboBox(self)
        self.language_selector.addItems(['English', 'Русский', '中文'])
        self.language_selector.currentIndexChanged.connect(self.change_language)
        layout.addWidget(self.language_selector)

        # Метка для файла
        self.file_label = QLabel('Select a file to copy', self)
        layout.addWidget(self.file_label)

        # Кнопка для выбора файла
        self.select_file_button = QPushButton('Select File', self)
        self.select_file_button.clicked.connect(self.select_file)
        layout.addWidget(self.select_file_button)

        # Лист для отображения папок с поддержкой drag-and-drop
        self.folders_list = QListWidget(self)
        self.folders_list.setAcceptDrops(True)
        self.folders_list.setDragEnabled(True)
        layout.addWidget(self.folders_list)

        # Прогресс-бар
        self.progress_bar = QProgressBar(self)
        layout.addWidget(self.progress_bar)

        # Кнопка для копирования файла
        self.copy_button = QPushButton('Copy File to Selected Folders', self)
        self.copy_button.clicked.connect(self.copy_file_to_folders)
        layout.addWidget(self.copy_button)

        # Кнопка для начала новой операции
        self.new_operation_button = QPushButton('Start New Operation', self)
        self.new_operation_button.clicked.connect(self.reset_operation)
        layout.addWidget(self.new_operation_button)

        self.setLayout(layout)
        self.setWindowTitle('File to Multiple Folders')

        self.file_path = ''
        self.folders = []
        self.language = 'English'

    def change_language(self):
        self.language = self.language_selector.currentText()
        if self.language == 'English':
            self.set_language_english()
        elif self.language == 'Русский':
            self.set_language_russian()
        elif self.language == '中文':
            self.set_language_chinese()

    def set_language_english(self):
        self.file_label.setText('Select a file to copy')
        self.select_file_button.setText('Select File')
        self.copy_button.setText('Copy File to Selected Folders')
        self.new_operation_button.setText('Start New Operation')

    def set_language_russian(self):
        self.file_label.setText('Выберите файл для копирования')
        self.select_file_button.setText('Выбрать файл')
        self.copy_button.setText('Копировать файл в выбранные папки')
        self.new_operation_button.setText('Начать новую операцию')

    def set_language_chinese(self):
        self.file_label.setText('选择要复制的文件')
        self.select_file_button.setText('选择文件')
        self.copy_button.setText('将文件复制到选定的文件夹')
        self.new_operation_button.setText('开始新操作')

    # Перетаскивание файлов в окно программы
    def dragEnterEvent(self, event):
        if event.mimeData().hasUrls():
            event.acceptProposedAction()

    # Обработка дропа файлов
    def dropEvent(self, event: QDropEvent):
        urls = event.mimeData().urls()
        for url in urls:
            folder_path = url.toLocalFile()
            if os.path.isdir(folder_path):
                self.folders_list.addItem(folder_path)
                self.folders.append(folder_path)

    # Функция для выбора файла
    def select_file(self):
        self.file_path, _ = QFileDialog.getOpenFileName(self, 'Select File')
        if self.file_path:
            self.file_label.setText(f'Selected file: {self.file_path}')

    # Функция для копирования файла в выбранные папки
    def copy_file_to_folders(self):
        if not self.file_path:
            self.file_label.setText('First, select a file!')
            return
        if not self.folders:
            self.file_label.setText('First, select folders!')
            return

        total_folders = len(self.folders)
        self.progress_bar.setMaximum(total_folders)

        for index, folder in enumerate(self.folders):
            try:
                destination_file = os.path.join(folder, os.path.basename(self.file_path))
                shutil.copy2(self.file_path, destination_file)
                self.progress_bar.setValue(index + 1)
            except Exception as e:
                self.file_label.setText(f'Error copying to {folder}: {e}')
                return
        
        self.file_label.setText('All files copied successfully!')

    # Функция для начала новой операции
    def reset_operation(self):
        self.file_path = ''
        self.folders.clear()
        self.folders_list.clear()
        self.progress_bar.setValue(0)
        self.file_label.setText('Select a file to copy')

if __name__ == '__main__':
    app = QApplication(sys.argv)
    ex = FileCopyApp()
    ex.setAcceptDrops(True)  # Разрешаем перетаскивание в окно
    ex.show()
    sys.exit(app.exec_())
