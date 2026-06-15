# 🇷🇺 Google Forms Test Generator / [🇬🇧 English below](#english)

---

## 🇷🇺 Русский

Скрипт автоматически создаёт тест в Google Forms из обычного текста с вопросами и ответами.

### 🚀 Что умеет

- Создаёт Google Форму с тестом
- Поддерживает один или несколько правильных ответов
- Проверяет ответы автоматически (режим викторины)
- Перемешивает варианты ответов при каждом создании формы
- Все вопросы помечаются как обязательные

### 📄 Формат текста

| Тег | Значение |
|-----|----------|
| `<q>` | Начало вопроса |
| `<a>` | Неправильный ответ |
| `<+a>` | Правильный ответ |

Если правильных ответов несколько — создаётся вопрос с чекбоксами. Если один — с радиокнопками.

**Пример:**

```
<q>Какой из редакторов является векторным?
<a>Adobe Photoshop
<+a>CorelDRAW
<a>Paint

<q>Выберите правильные ответы:
<+a>Правильный 1
<+a>Правильный 2
<a>Неправильный
```

### ⚙️ Как использовать

1. Зайди на [script.google.com](https://script.google.com/)
2. Создай новый проект
3. Вставь код из `code.gs`
4. В переменной `rawText` напиши свои вопросы в формате выше
5. Выбери функцию `myFunction` и нажми ▶ **Выполнить**
6. В логах появится ссылка на форму

### 🛠 Как это работает

- `rawText` — строка с вопросами и ответами
- Парсер разбирает текст, выделяет вопросы и ответы, отмечает правильные
- Ответы перемешиваются через `shuffleArray` перед добавлением в форму
- Каждый вопрос помечается как обязательный (`.setRequired(true)`)
- `myFunction` — точка входа, вызывает `createTestForm`

> ⚠️ **Примечание:** галочка «Перемешать ответы» в интерфейсе Google Forms недоступна через API. Перемешивание происходит на этапе создания формы через `shuffleArray`.

### 📂 Структура проекта

```
google-forms-test-generator/
├── README.md   # описание и инструкция
└── code.gs     # основной скрипт
```

### 💻 Код

```javascript
function createTestForm() {
  const form = FormApp.create('Название вашего теста');
  form.setIsQuiz(true);
  
  const rawText = `
<q>Пример вопроса 1?
<a>Ответ 1
<+a>Ответ 2
<a>Ответ 3

<q>Вопрос с несколькими правильными ответами
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
      item.setTitle(questionText).setPoints(1).setRequired(true);
      const choices = answers.map(a => item.createChoice(a.text, a.correct));
      item.setChoices(choices);
    } else {
      const item = form.addMultipleChoiceItem();
      item.setTitle(questionText).setPoints(1).setRequired(true);
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

// Точка входа — запускай именно эту функцию
function myFunction() {
  createTestForm();
}
```

### 🤖 Промт для ИИ — конвертация любого материала в формат теста

Если у тебя есть параграф из учебника, конспект, статья или любой другой текст — скопируй промт ниже и вставь его в ChatGPT, Claude или любой другой ИИ вместе со своим материалом.

````
Ты — генератор тестов. Твоя задача — преобразовать предоставленный текст в формат теста для скрипта Google Forms.

Правила формата:
- Каждый вопрос начинается с тега <q>
- Неправильный ответ помечается тегом <a>
- Правильный ответ помечается тегом <+a>
- Если вопрос предполагает несколько правильных ответов — используй несколько тегов <+a>
- У каждого вопроса должно быть от 3 до 5 вариантов ответа
- Ответы должны быть правдоподобными и примерно одинаковой длины
- Не добавляй никаких пояснений, только готовый текст в формате

Пример правильного вывода:
<q>Какой тег используется для обозначения правильного ответа?
<a><q>
<+a><+a>
<a><a>
<a><-a>

<q>Сколько правильных ответов может быть у одного вопроса?
<+a>Один или несколько
<a>Только один
<a>Не более двух
<a>Ровно три

Теперь преобразуй следующий текст в тест (минимум 10 вопросов):

[ВСТАВЬ СВОЙ ТЕКСТ СЮДА]
````

> 💡 **Совет:** чем подробнее исходный текст — тем качественнее получится тест. Можно вставлять целые главы из учебников, конспекты лекций, статьи из Википедии.

---

<a name="english"></a>
## 🇬🇧 English

A script that automatically generates a Google Forms quiz from plain text with questions and answers.

### 🚀 Features

- Creates a Google Form quiz
- Supports single and multiple correct answers
- Auto-grades answers (quiz mode)
- Shuffles answer choices on every form creation
- All questions are marked as required

### 📄 Text Format

| Tag | Meaning |
|-----|---------|
| `<q>` | Question start |
| `<a>` | Wrong answer |
| `<+a>` | Correct answer |

Multiple correct answers → checkbox question. Single correct answer → multiple choice.

**Example:**

```
<q>Which editor is vector-based?
<a>Adobe Photoshop
<+a>CorelDRAW
<a>Paint

<q>Select all correct answers:
<+a>Correct 1
<+a>Correct 2
<a>Wrong
```

### ⚙️ How to Use

1. Go to [script.google.com](https://script.google.com/)
2. Create a new project
3. Paste the code from `code.gs`
4. Write your questions in `rawText` using the format above
5. Select `myFunction` and click ▶ **Run**
6. The form link will appear in the logs

### 🛠 How It Works

- `rawText` — string containing all questions and answers
- Parser splits questions, extracts answers, marks correct ones
- Answers are shuffled via `shuffleArray` before being added to the form
- Every question is set as required (`.setRequired(true)`)
- `myFunction` is the entry point that calls `createTestForm`

> ⚠️ **Note:** The "Shuffle answers" checkbox in Google Forms UI is not accessible via the Apps Script API. Shuffling happens at form creation time via `shuffleArray`.

### 📂 Project Structure

```
google-forms-test-generator/
├── README.md   # description and instructions
└── code.gs     # main script
```

### 🤖 AI Prompt — Convert Any Document to Test Format

Have a textbook paragraph, lecture notes, or any material? Copy the prompt below and paste it into ChatGPT, Claude, or any other AI along with your content.

````
You are a test generator. Your task is to convert the provided text into a quiz format for the Google Forms script.

Format rules:
- Each question starts with the <q> tag
- Wrong answers are marked with the <a> tag
- Correct answers are marked with the <+a> tag
- If a question has multiple correct answers — use multiple <+a> tags
- Each question must have 3 to 5 answer options
- Answers should be plausible and roughly the same length
- Output only the formatted text, no explanations

Example of correct output:
<q>Which tag marks a correct answer?
<a><q>
<+a><+a>
<a><a>
<a><-a>

<q>How many correct answers can a question have?
<+a>One or more
<a>Exactly one
<a>No more than two
<a>Exactly three

Now convert the following text into a quiz (minimum 10 questions):

[PASTE YOUR TEXT HERE]
````

> 💡 **Tip:** the more detailed your source text, the better the quiz. You can paste full textbook chapters, lecture notes, or Wikipedia articles.
