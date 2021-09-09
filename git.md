Настройка git репозитария
-  Устанавливаем git ``` yum install git```
-  Проверяем установку и версию
```
git --version
#git version 2.27.0
```
- создаем папку под репозиторий ```mkdir git_repo```
- переходим в созданную папку ```cd git_repo```
- включаем git для данной папки ```git init```
- делаем первоначальную настройку git 
```
git config --global user.name "otus demo" # установка какой user будет работать с git
git config --global user.email a.trubin@ulnanotech.com
git config --global core.editor nano # установка редактора по умолчанию
```
- для индексации файлов и создание комита 
```
git add .
git commit -m "first commit"
```
- для удаленного сохранения репозитория формируем ключ ```ssh-keygen``` и ищем его ```~/.ssh/```
- на Github в ```Account settings -> SSH and GPG keys``` прописываем наш SSH ключ
- копируем ссылку для ssh доступа ```git@github.com:Flydog7676/otus_ege.git```
- делаем клон репозитария в папку ``` git clone git@github.com:Flydog7676/otus_ege.git``` 
