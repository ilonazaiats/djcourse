
# Web Scraping. Я взяла сайт zaxid.net, де знаходяться статті про вуличне освітлення Львова (якраз для фінального проєкту)

# Імпортуємо необхідні бібліотеки:
```{r setup, include=FALSE}
library(rvest)
library(tidyverse)
library(progress)
```

# Отримуємо html
```{r}
url <- "https://zaxid.net/vulichne_osvitlennya_tag43706/"
content <- read_html(url)
content
```

# Знаходимо потрібні елементи на сторінці з допомогою функції inspect
# Спробуємо вибрати рядки з таблиці
```{r}
content %>%
  html_nodes("div.news-title") %>%
  html_text()
```

# Виберемо лише заголовки та дати, використаємо для цього CSS-селектори за допомогою атрибутів.
```{r}
titles <- content %>%
  html_nodes('div.news-title') %>%
  html_text() %>%
  str_trim()
titles 

dates <- content %>%
  html_nodes('div.time') %>%
  html_text() %>%
  str_trim()

dates
```

# Створюємо та зберігаємо таблицю
```{r}
df <- data.frame(titles = titles, date = dates)

write.csv(df, "eu_disinfo_1st_page.csv", row.names = FALSE)
```

# Читаємо csv
```{r}
read.csv("eu_disinfo_1st_page.csv")
```

# Зациклюємо та качаємо те саме для кожної сторінки
```{r}
npages <- 2

pb <- progress_bar$new(
  format = "  downloading [:bar] :percent in :elapsed ",
  total = npages, clear = FALSE, width= 60)

# Вектори, у яких будемо зберігати значення
dates <- c()
titles <- c()
links <- c()

url_template <- "https://zaxid.net/vulichne_osvitlennya_tag43706/newsfrom"
```

# Оскільки в цьому розділі новин є лише 2 сторінки, то зациклюємо їх.
```{r}
for (page in 1:npages) {
  # з'єднуємо рядки: основу url-адреси, № сторінки помножений на 10, бо сторінки йдуть з кроком 10
  url <- str_c(url_template,
               page * 44)
  
  content <- read_html(url)
  
  # Копіпаст коду для першої сторінки
  titles <- content %>%
    html_nodes('div.news-title') %>%
    html_text() %>%
    str_trim() %>%
    c(titles, .)    # "." крапка означає змінну в пайпі, якщо пона не на першому місці 
  
  dates <- content %>%
    html_nodes('div.time') %>%
    html_text() %>%
    str_trim()  %>%
    c(dates, .)
  
  # ще додамо лінки. Тут вибираємо не текст, а атрибут "href" тега "<a>" — лінк
  links <- content %>%
    html_nodes('div.news-title a') %>%
    html_attr("href") %>%
    c(links, .)
  
  # Ще один важливий крок: затримка між запитами, щоб не зробити DDoS-атаку на сайт
  Sys.sleep(3)    # 2 секунди програма буде "спати" 
  
  # Оновимо прогрес-бар. Це для комфорту, щоб бачити, скільки ще лишилось сторінок
  pb$tick()
}
```