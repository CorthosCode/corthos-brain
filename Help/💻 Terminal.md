

![](assets/-o7s0jcmuZXr3VeJV9BCagxq9uySAosXEuJ8-4gPO5o=.png)


## Шаг 1: установить Oh My ZSH

curl:

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

или wget:

```sh
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

Инструкцию можно найти на оф. сайте: [https://ohmyz.sh/](https://ohmyz.sh/#install)


## Шаг 2: установить тему

Необходимо открыть файл `~/.zshrc` и поправить его в зависимости от выбранной темы.

Все темы можно найти здесь (https://github.com/ohmyzsh/ohmyzsh/wiki/Themes).

`sudo nano ~/.zshrc`

Нас интересует поле ZSH\_THEME="robbyrussell", я его меняю на значение `agnoster`

`ZSH_THEME="agnoster"`

Сохраняем файл и выходим из редактора.

Для того чтобы изменения вступили в силу - перезагрузить терминал.


## Шаг 3: установить Powerline шрифт

В репозитории (https://github.com/powerline/fonts) со шрифтами есть несколько способов с установкой, но мне показался самый простой тот, где необходимо загрузить исходные файлы и выполнить `./install.sh`

```sh
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
```


## Шаг 4: установить шрифт в терминале

Необходимо зайти в настройки терминала, и во вкладке `Профили` выбрать добавленный нами новый шрифт нажав на кнопку `Изменить...`. У меня это выглядит так:

![](assets/Wp_EolGigE-k-RlnsXMgLdL_fuf00dUaNRGBfk8Wx5c=.jpeg)


## Шаг 5: установить цветовую тему

С цветами можно поиграться самому, но удобнее всего использовать готовые пресеты, которые уже хорошо подобраны. Вот ссылка (https://github.com/lysyi3m/macos-terminal-themes) на все цветовые темы, необходимо просто скачать тему нажав `Сохранить как`.

В настройках терминала слева в окне нажать на кнопку `Импортирвать` и выбрать скачанную тему. А также нажать `По умолчанию`.

![](assets/RNKzjQI9GGbfSud2TIeEB2CCDJ1Mv-a15g0ZFt6egOI=.png)


***

Исходный материал находится здесь:

https://dev.to/hannahgooding/how-i-customise-my-terminal-with-oh-my-zsh-macos-427i

