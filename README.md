# Google-Forms-Test-Generator

Скрипт, который автоматически создаёт тест в Google Forms из обычного текста с вопросами и ответами.

## 🚀 Что умеет
- Делает форму с тестом  
- Поддерживает один или несколько правильных ответов  
- Проверяет ответы автоматически  
- Перемешивает варианты  

## 📄 Формат текста
- Вопрос начинается с `<q>`  
- Неправильный ответ помечается `<a>`  
- Правильный ответ — `<+a>`  
- Если правильных несколько, получится выбор с галочками  

**Пример:**
<q>Какой из указанных графических редакторов является векторным?
<a>CorelDRAW
<a>Adobe Photoshop
<+a>Paint


## ⚙️ Как пользоваться

1. Зайди в [Google Apps Script](https://script.google.com/).  
2. Создай новый проект и вставь туда код из файла `code.gs`.  
3. В переменной `rawText` напиши свои вопросы в формате выше.  
4. Запусти функцию `createTestForm`.  
5. Скрипт создаст Google Форму с тестом.  
6. В логах появится ссылка для редактирования формы.  

## 🛠 Как это работает
- `rawText` — строка с вопросами и ответами.  
- Парсер разбирает текст, выделяет вопросы и ответы, отмечает правильные.  
- Если правильных несколько, создаётся вопрос с галочками. Если один — с кружочками.  
- Варианты ответов перемешиваются.  
- Для работы используется `FormApp`.  

## 🔧 Как расширить
- Добавить поддержку других типов вопросов, например, текстовых.  
- Изменить количество баллов за вопрос (по умолчанию 1).  
- Настроить оформление формы: описание, разделы и так далее.  

## 📂 Структура проекта
google-forms-test-generator/
│── README.md # инструкция и описание
│── code.gs # основной скрипт


## 💻 Код скрипта

```javascript
function createTestForm() {
  const form = FormApp.create('Название вашего теста');
  form.setIsQuiz(true);
  
  const rawText = `
<q>Пример вопроса 1?
<a>Ответ 1
<+a>Ответ 2
<a>Ответ 3

<q>Пример вопроса 2 с несколькими правильными ответами
<+a>Правильный ответ 1
<+a>Правильный ответ 2
<a>Неправильный ответ
`;

  const questions = rawText.trim().split(/<q>/).filter(Boolean);

  for (let q of questions) {
    let lines = q.trim().split('\n').filter(line => line.trim() !== '');
    if (lines.length === 0) continue;

    let questionText = lines[0].trim();
    let answers = [];

    for (let i = 1; i < lines.length; i++) {
      const line = lines[i].trim();
      if (line.startsWith('<+a>')) {
        answers.push({ text: line.replace('<+a>', '').trim(), correct: true });
      } else if (line.startsWith('<a>')) {
        answers.push({ text: line.replace('<a>', '').trim(), correct: false });
      }
    }

    answers = shuffleArray(answers);

    if (answers.filter(a => a.correct).length > 1) {
      const item = form.addCheckboxItem();
      item.setTitle(questionText).setPoints(1);
      const choices = answers.map(a => item.createChoice(a.text, a.correct));
      item.setChoices(choices);
    } else {
      const item = form.addMultipleChoiceItem();
      item.setTitle(questionText).setPoints(1);
      const choices = answers.map(a => item.createChoice(a.text, a.correct));
      item.setChoices(choices);
    }
  }

  Logger.log('Форма создана: ' + form.getEditUrl());
}

function shuffleArray(array) {
  const arr = array.slice();
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
}
