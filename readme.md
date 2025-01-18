
# Создайте интерактивную веб-страницу для оставления и просмотра отзывов о продуктах. Пользователи могут добавлять отзывы о различных продуктах и просматривать добавленные отзывы.

## Страница добавления отзыва:

Поле для ввода названия продукта. \
Текстовое поле для самого отзыва. \
Кнопка "Добавить отзыв", которая сохраняет отзыв о продукте в LocalStorage.

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Добавить отзыв</title>
</head>
<body>
    <h1>Добавить отзыв:</h1>
    <input type="text" id="product">
    <input type="text" id="feedback">
    <button class="add-feedback">Добавить отзыв</button>

    <script>
        const productInp = document.querySelector('#product');
        const feedbackInp = document.querySelector('#feedback');
        const addFeedbackBtn = document.querySelector('.add-feedback');

        addFeedbackBtn.addEventListener('click', function (e) {
            if (productInp.value !== '' && feedbackInp.value !== '') { // Если заполнен продукт и отзыв о нем
                 // Читаем из localStorage отзывы о продуктах
                const feedbacksStr = localStorage.getItem('feedbacks');
                // Если ничего не прочитано создаем пустой объект отзывов
                const feedbacks = feedbacksStr ? JSON.parse(feedbacksStr) : {};
                 // Дополняем отзывы новым отзывом
                feedbacks[productInp.value] = feedbacks[productInp.value] 
                                              ? [...feedbacks[productInp.value], feedbackInp.value] 
                                              : [feedbackInp.value];
                // Пишем в localStorage отзывы о продуктах
                localStorage.setItem('feedbacks', JSON.stringify(feedbacks));
                 // Переходим на страницу отзывов
                window.location.href = './feedbacks.html';
            }
        });

    </script>
</body>
</html>
```

## Страница просмотра отзывов:

Показывает список всех продуктов, о которых были оставлены отзывы. \
При клике на название продукта отображается список всех отзывов по этому продукту. \
Возможность удаления отзыва (при нажатии на кнопку "Удалить" рядом с отзывом, данный отзыв удаляется из LocalStorage).

```
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Отзывы</title>
        <link rel="stylesheet" href="./feedbacks.css">
    </head>
    <body>
        <h1>Отзывы</h1>
        <a href="./index.html">Новый отзыв</a>
        <div class="content"></div>

        <script>
            const contentEl = document.querySelector(".content");

            function createEl(rootEl, typeEl, textContenEl, classEl = [], attributesEl = {}) {
                const newEl = document.createElement(typeEl);
                newEl.textContent = textContenEl;

                classEl.forEach(className => newEl.classList.add(className));

                for (const attr in attributesEl) newEl.setAttribute(attr, attributesEl[attr]);

                rootEl.append(newEl);
                return newEl;
            }

            window.addEventListener("load", function (e) {
                const feedbacksStr = localStorage.getItem("feedbacks");
                if (feedbacksStr) {
                    const feedbacks = JSON.parse(feedbacksStr);

                    const newProductList = createEl(contentEl, 'ul', 'Список продуктов с отзывами: ')
                    for (const product in feedbacks) createEl(newProductList, "li", product, ["product"]);

                } else {
                    contentEl.textContent = "Нет ни одного отзыва";
                    alert("Нет ни одного отзыва! Сначала создайте хоть один отзыв!");
                    window.location.href = "./index.html";
                }
            });

            contentEl.addEventListener("click", function (e) {
                const feedbacksStr = localStorage.getItem("feedbacks");
                if (!feedbacksStr) {
                    return;
                }
                const feedbacks = JSON.parse(feedbacksStr);

                if (e.target.classList.contains("product")) { // Нажат элемент списка с продуктом
                    const productName = e.target.textContent;

                    for (const product in feedbacks) {
                        if (product === productName) {
                            // Очищаем от старого списка с отзывами, если он есть
                            contentEl.querySelectorAll(".feedbacks").forEach(el => el.remove());

                            // Создаем новый список с отзывами
                            const feedbacksUl = createEl(contentEl,
                                                        'ul',
                                                        `Отзывы о продукте '${productName}':`,
                                                        ["feedbacks"]);

                            // Создаем отзывы и кнопки удаления отзывов
                            for (let index = 0; index < feedbacks[product].length; index++) {
                                const feedbackLi = createEl(feedbacksUl, "li", feedbacks[product][index]);
                                const newFeedbackBtn = createEl(feedbackLi,
                                                                "button",
                                                                "-",
                                                                ["delete-feedback"],
                                                                {
                                                                    'data-number': index,
                                                                    'data-product': productName
                                                                }); // Нумеруем кнопки для удаления отзывов
                            }

                            break;
                        }
                    }

                } else if (e.target.classList.contains("delete-feedback")) { // Нажата кнопка удаления отзыва

                    const feedbackNumber = e.target.getAttribute("data-number");
                    const productName = e.target.getAttribute("data-product");

                    // Удаляем отзыв из объекта "отзывы" (feedbacks)
                    feedbacks[productName].splice(feedbackNumber, 1);

                    // Пишем результат в локальное хранилище
                    localStorage.setItem('feedbacks', JSON.stringify(feedbacks));

                    // Получаем список ul в котором расположены отзывы
                    const parentUl = e.target.parentElement.parentElement;

                    // Удаляем элемент списка, связанный с отзывом из HTML
                    e.target.parentElement.remove();

                    // Перенумеруем кнопки в соответствии с новым состоянием объекта "отзывы" (feedbacks)
                    const deleteBtns = parentUl.querySelectorAll('button');
                    for (let index = 0; index < deleteBtns.length; index++) {
                        deleteBtns[index].setAttribute("data-number", index)
                    }

                }
            });
        </script>
    </body>
</html>
```
