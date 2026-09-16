<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mini Pinterest</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    background: #f5f5f5;
    color: #111;
  }

  /* ---------- Шапка ---------- */
  header {
    position: sticky;
    top: 0;
    z-index: 100;
    background: #fff;
    padding: 14px 20px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.06);
    display: flex;
    align-items: center;
    gap: 16px;
  }

  .logo {
    font-size: 22px;
    font-weight: 700;
    color: #e60023;
    white-space: nowrap;
  }

  .search {
    flex: 1;
    padding: 12px 18px;
    border-radius: 24px;
    border: none;
    background: #efefef;
    font-size: 15px;
    outline: none;
    transition: background 0.2s;
  }
  .search:focus { background: #e2e2e2; }

  .add-btn {
    background: #e60023;
    color: #fff;
    border: none;
    padding: 12px 20px;
    border-radius: 24px;
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
    white-space: nowrap;
    transition: background 0.2s;
  }
  .add-btn:hover { background: #ad081b; }

  /* ---------- Сетка (masonry через columns) ---------- */
  .grid {
    column-count: 4;
    column-gap: 16px;
    padding: 20px;
    max-width: 1600px;
    margin: 0 auto;
  }

  @media (max-width: 1200px) { .grid { column-count: 3; } }
  @media (max-width: 800px)  { .grid { column-count: 2; } }
  @media (max-width: 500px)  { .grid { column-count: 1; } }

  /* ---------- Карточка ---------- */
  .card {
    break-inside: avoid;
    margin-bottom: 16px;
    border-radius: 16px;
    overflow: hidden;
    position: relative;
    background: #ddd;
    cursor: pointer;
    transition: transform 0.2s, box-shadow 0.2s;
  }
  .card:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 20px rgba(0,0,0,0.15);
  }

  .card img {
    width: 100%;
    display: block;
  }

  /* Оверлей с кнопкой лайка */
  .overlay {
    position: absolute;
    inset: 0;
    background: rgba(0,0,0,0.4);
    opacity: 0;
    transition: opacity 0.2s;
    display: flex;
    justify-content: flex-end;
    align-items: flex-start;
    padding: 12px;
  }
  .card:hover .overlay { opacity: 1; }

  .like-btn {
    background: #fff;
    border: none;
    width: 40px;
    height: 40px;
    border-radius: 50%;
    font-size: 18px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: transform 0.15s, background 0.2s;
  }
  .like-btn:hover { transform: scale(1.1); background: #ffe0e0; }
  .like-btn.liked { background: #e60023; color: #fff; }

  .delete-btn {
    position: absolute;
    top: 12px;
    left: 12px;
    background: #fff;
    border: none;
    width: 40px;
    height: 40px;
    border-radius: 50%;
    font-size: 16px;
    cursor: pointer;
    opacity: 0;
    transition: opacity 0.2s;
  }
  .card:hover .delete-btn { opacity: 1; }

  .card-title {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    padding: 24px 12px 12px;
    background: linear-gradient(transparent, rgba(0,0,0,0.7));
    color: #fff;
    font-size: 14px;
    font-weight: 500;
    opacity: 0;
    transition: opacity 0.2s;
  }
  .card:hover .card-title { opacity: 1; }

  /* ---------- Пустое состояние ---------- */
  .empty {
    text-align: center;
    padding: 80px 20px;
    color: #888;
    font-size: 18px;
    column-span: all;
  }

  /* ---------- Модалка добавления ---------- */
  .modal {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.5);
    display: none;
    align-items: center;
    justify-content: center;
    z-index: 200;
    padding: 20px;
  }
  .modal.open { display: flex; }

  .modal-content {
    background: #fff;
    border-radius: 20px;
    padding: 28px;
    width: 100%;
    max-width: 420px;
  }
  .modal-content h2 { margin-bottom: 20px; font-size: 22px; }

  .modal-content input {
    width: 100%;
    padding: 12px 16px;
    border: 1px solid #ddd;
    border-radius: 10px;
    font-size: 15px;
    margin-bottom: 12px;
    outline: none;
  }
  .modal-content input:focus { border-color: #e60023; }

  .modal-actions {
    display: flex;
    gap: 10px;
    margin-top: 8px;
  }
  .modal-actions button {
    flex: 1;
    padding: 12px;
    border: none;
    border-radius: 24px;
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
  }
  .btn-cancel { background: #efefef; color: #111; }
  .btn-save { background: #e60023; color: #fff; }
  .btn-save:hover { background: #ad081b; }
</style>
</head>
<body>

<header>
  <div class="logo">📌 MiniPin</div>
  <input class="search" id="search" type="text" placeholder="Поиск по названию...">
  <button class="add-btn" id="openModal">+ Добавить</button>
</header>

<div class="grid" id="grid"></div>

<!-- Модалка -->
<div class="modal" id="modal">
  <div class="modal-content">
    <h2>Новая картинка</h2>
    <input id="imgUrl" type="text" placeholder="Ссылка на картинку (URL)">
    <input id="imgTitle" type="text" placeholder="Название">
    <div class="modal-actions">
      <button class="btn-cancel" id="cancelBtn">Отмена</button>
      <button class="btn-save" id="saveBtn">Добавить</button>
    </div>
  </div>
</div>

<script>
// ---------- Стартовые данные ----------
const defaultPins = [
  { id: 1, url: 'https://picsum.photos/id/1015/400/600',  title: 'Горная река' },
  { id: 2, url: 'https://picsum.photos/id/1016/400/500',  title: 'Лес' },
  { id: 3, url: 'https://picsum.photos/id/1018/400/700',  title: 'Водопад' },
  { id: 4, url: 'https://picsum.photos/id/1024/400/450',  title: 'Кот' },
  { id: 5, url: 'https://picsum.photos/id/1035/400/650',  title: 'Море' },
  { id: 6, url: 'https://picsum.photos/id/1043/400/550',  title: 'Дорога' },
  { id: 7, url: 'https://picsum.photos/id/1050/400/600',  title: 'Горы' },
  { id: 8, url: 'https://picsum.photos/id/1062/400/400',  title: 'Пляж' },
];

// ---------- Состояние (localStorage) ----------
let pins = JSON.parse(localStorage.getItem('pins')) || defaultPins;
let likes = JSON.parse(localStorage.getItem('likes')) || [];
let query = '';

const grid = document.getElementById('grid');
const search = document.getElementById('search');
const modal = document.getElementById('modal');

// ---------- Отрисовка ----------
function render() {
  const filtered = pins.filter(p =>
    p.title.toLowerCase().includes(query.toLowerCase())
  );

  if (filtered.length === 0) {
    grid.innerHTML = '<div class="empty">Ничего не найдено 🔍</div>';
    return;
  }

  grid.innerHTML = filtered.map(pin => `
    <div class="card" data-id="${pin.id}">
      <img src="${pin.url}" alt="${pin.title}" loading="lazy">
      <div class="overlay">
        <button class="like-btn ${likes.includes(pin.id) ? 'liked' : ''}"
                data-like="${pin.id}">
          ${likes.includes(pin.id) ? '❤️' : '🤍'}
        </button>
      </div>
      <button class="delete-btn" data-del="${pin.id}">🗑️</button>
      <div class="card-title">${pin.title}</div>
    </div>
  `).join('');
}

// ---------- Сохранение ----------
function save() {
  localStorage.setItem('pins', JSON.stringify(pins));
  localStorage.setItem('likes', JSON.stringify(likes));
}

// ---------- Делегирование событий ----------
grid.addEventListener('click', (e) => {
  const likeId = e.target.dataset.like;
  const delId  = e.target.dataset.del;

  // Лайк
  if (likeId) {
    e.stopPropagation();
    const id = Number(likeId);
    likes = likes.includes(id)
      ? likes.filter(x => x !== id)
      : [...likes, id];
    save();
    render();
    return;
  }

  // Удаление
  if (delId) {
    e.stopPropagation();
    const id = Number(delId);
    if (confirm('Удалить картинку?')) {
      pins = pins.filter(p => p.id !== id);
      likes = likes.filter(x => x !== id);
      save();
      render();
    }
  }
});

// ---------- Поиск ----------
search.addEventListener('input', (e) => {
  query = e.target.value;
  render();
});

// ---------- Модалка ----------
document.getElementById('openModal').onclick = () => modal.classList.add('open');
document.getElementById('cancelBtn').onclick = () => modal.classList.remove('open');

document.getElementById('saveBtn').onclick = () => {
  const url = document.getElementById('imgUrl').value.trim();
  const title = document.getElementById('imgTitle').value.trim() || 'Без названия';

  if (!url) return alert('Вставь ссылку на картинку');

  pins.unshift({ id: Date.now(), url, title });
  save();
  render();

  document.getElementById('imgUrl').value = '';
  document.getElementById('imgTitle').value = '';
  modal.classList.remove('open');
};

// Закрытие по клику вне модалки
modal.addEventListener('click', (e) => {
  if (e.target === modal) modal.classList.remove('open');
});

// ---------- Старт ----------
render();
</script>
</body>
</html>
