# ![joomla](joomla.png)Ansible LEMP Stack с Joomla CMS

Этот Ansible проект автоматизирует развертывание LEMP-стека (Linux, Nginx, MySQL, PHP) с CMS Joomla на серверах Ubuntu.

## Требования

- Ansible 2.9+
- Сервер Ubuntu 20.04/22.04
- Доступ по SSH к целевому серверу с правами sudo
- Python 3.x на целевом сервере

## Установка

1. Клонируйте репозиторий:

   ```
   git clone https://github.com/Perovss/rebrain-ansible.git
   cd rebrain-ansible
   ```

2. Настройте инвентарь:

   - Отредактируйте `inventory.yml` с данными вашего сервера
   - Убедитесь, что ваш SSH-ключ настроен для аутентификации

3. Настройте переменные:

   - Отредактируйте `group_vars/all/vars.yml` для базовой конфигурации

   - Для чувствительных данных отредактируйте vault-файл:

     ```
     ansible-vault edit group_vars/all/vault.yml
     ```

     Установите эти переменные:

     ```
     vault_mysql_root_password: "ваш_root_пароль_mysql"
     vault_mysql_user_password: "ваш_пароль_пользователя_mysql"
     ```

4. Установите необходимые роли Ansible:

   ```
   ansible-galaxy install -r requirements.yml
   ```

## Развертывание

Запустите плейбук:

```
ansible-playbook playbook.yml --ask-vault-pass
```

Вам будет предложено ввести vault-пароль, который вы установили при шифровании vault-файла.

## Настройка после установки

После успешного развертывания:

1. Откройте IP-адрес сервера в веб-браузере
2. Завершите установку Joomla через веб-интерфейс:
   - Название сайта: Укажите имя вашего сайта
   - Учетные данные суперпользователя: Установите имя пользователя/пароль администратора
   - Конфигурация базы данных:
     - Тип базы данных: MySQLi
     - Имя хоста: localhost
     - Имя пользователя: admin (или ваш настроенный пользователь MySQL)
     - Пароль: [используйте пароль пользователя MySQL из vault]
     - Имя базы данных: joomla
     - Префикс таблиц: jml_ (или оставьте по умолчанию)
3. Удалите папку installation, когда будет предложено

## Детали конфигурации

### Настройки по умолчанию

- Корневая директория сайта: `/var/www/sites/joomla`
- Конфиг Nginx: `/etc/nginx/sites-available/joomla.conf`
- Версия PHP: 8.1
- MySQL:
  - Пароль root: Установлен в vault
  - Пользователь приложения: admin (пароль установлен в vault)
  - Имя базы данных: joomla

### Версия Joomla

Плейбук по умолчанию устанавливает Joomla {{ joomla_version }}. Для изменения:

1. Отредактируйте `roles/joomla/vars/main.yml`
2. Обновите номер версии
3. Перезапустите плейбук

## Примечания по безопасности

1. После установки:
   - Настройте SSL/TLS для Nginx
   - Настройте брандмауэр (ufw)
   - Рассмотрите возможность изменения стандартных портов MySQL
   - Настройте регулярное резервное копирование
2. Зашифрованные vault-пароли защищены только настолько, насколько надежен ваш vault-пароль. Регулярно меняйте пароли.

## Обслуживание

### Обновление Joomla

1. Сделайте резервную копию сайта и базы данных
2. Скачайте последний пакет Joomla
3. Следуйте официальной процедуре обновления Joomla

### Обновление плейбука

Для изменения конфигурации:

1. Обновите переменные соответствующих ролей
2. Протестируйте изменения в тестовой среде
3. Перезапустите плейбук с `--tags` для конкретных компонентов при необходимости

## Устранение неполадок

Частые проблемы:

1. **Страница установки отображается после настройки**:

   ```
   rm -rf /var/www/sites/joomla/installation
   ```

2. **Проблемы с правами доступа**:

   ```
   chown -R www-data:www-data /var/www/sites/joomla
   find /var/www/sites/joomla -type d -exec chmod 755 {} \;
   find /var/www/sites/joomla -type f -exec chmod 644 {} \;
   ```

3. **Ошибки Nginx/PHP-FPM**:
   Проверьте логи:

   ```
   journalctl -u nginx -u php8.1-fpm --no-pager -n 50
   ```

   