
# Automatic OpenWRT builds for Xiaomi Redmi AX6S (AX3200)

[![LICENSE](https://img.shields.io/github/license/mashape/apistatus.svg?style=flat-square&label=LICENSE)](https://github.com/P3TERX/Actions-OpenWrt/blob/master/LICENSE)


# Инструкция по установке для Windows (гайд от [stroti](https://4pda.to/forum/index.php?showuser=4540991))
1. Скачиваем и устанавливаем через стандартное обновление роутера dev версию официальной прошивки [miwifi_rb03_firmware_stable_1.2.7.bin](https://raw.githubusercontent.com/YangWang92/AX6S-unlock/master/miwifi_rb03_firmware_stable_1.2.7.bin) (MD5: 5eedf1632ac97bb5a6bb072c08603ed7).

2. Скачиваем и устанавливаем [Python](https://www.python.org/ftp/python/3.10.10/python-3.10.10-amd64.exe)

    2.1 Скачиваем [скрипт](https://raw.githubusercontent.com/mikeeq/xiaomi_ax3200_openwrt/main/password.py) перейдя по ссылке и кликнув "Сохранить как" password.py

    2.2 Запускаем ранее установленный IDLE (Python 3.10 64-bit) >> File > Open > password.py в открывшимся окне жмем Shift+F5 > в поле ввода вставляем серийный номер роутера и жмем Ok. На выходе получаем пароль от Telnet

3. Скачиваем программу [PuTTY](https://www.putty.org/)

4. Запускаем PuTTY, в адресную строку вбиваем 192.168.31.1, в выпадающем списке под портом выбираем Telnet и жмем Open

    4.1. Откроется консоль, в поле логина вводим root в поле пароль вставляем ранее полученный пароль из скрипта (ввод символов в пароле не отображается)
    
    4.2. Копируем и вставляем следующие команды (Можно последовательно, можно скопом, если будите вставлять скопом не забудьте нажать Enter для выполнения последней в списке команды)

    ```
    nvram set ssh_en=1
    nvram set uart_en=1
    nvram set boot_wait=on
    nvram commit
    sed -i '/flg_ssh.*release/ { :a; N; /fi/! ba };/return 0/d' /etc/init.d/dropbear
    /etc/init.d/dropbear enable
    /etc/init.d/dropbear start
    ```


5. Скачиваем программу [WinSCP](https://winscp.net/eng/download.php)
    
    5.1. Запускаем 
    
    5.2 Протокол передачи выбираем SCP, Имя хоста: 192.168.31.1 Имя пользователя:  root Пароль: сгенерированный из скрипта
    
    5.3. В правой части программы поднимаемся в корневой каталог нажав на пиктограмму папки с стрелкой, после чего находим папку /tmp и переходим в нее
    
    5.4. Скачиваем прошивку [OpenWRT](https://github.com/kyoto44/openwrt-ax6s/releases), а именно файл openwrt-mediatek-mt7622-xiaomi_redmi-router-ax6s-squashfs-factory.bin и сохраняем ее как factory.bin

    5.5. Перетаскиваем наш factory.bin в открытую на WinSCP папку /tmp и дожидаемся загрузки

6. Заново открываем PyTTY в адресную строку вбиваем 192.168.31.1 (протокол в этот раз по умолчанию SSH) и жмем Open
    
    6.1. Логин root, пароль ранее полученный из скрипта (ввод символов в пароле не отображается)
    
    6.2. Вводим следующие команды (последняя команда запускает процесс прошивки, дороги назад уже не будет, будьте внимательнее)
    ```
    nvram set flag_boot_success=1
    nvram set flag_try_sys1_failed=0
    nvram set flag_try_sys2_failed=0
    nvram commit

    cd /tmp
    mtd -r write factory.bin firmware
    ```

    Если после ввода этих комманд роутер (RB03, AX6S) перегрузился со стоковой прошивкой, то используем повторяем пункт 6 со следующими комманды:

    ```
    nvram set flag_boot_rootfs=0
    nvram set "boot_fw1=run boot_rd_img;bootm"
    nvram set flag_boot_success=1
    nvram set flag_try_sys1_failed=0
    nvram set flag_try_sys2_failed=0
    nvram commit

    cd /tmp
    mtd -r write factory.bin firmware
    ```

7. Сидим и ждем, после прошивки роутер сам перезагрузится, а новая прошивка OpenWRT запустится по адресу 192.168.1.1 логин root пароль по молчанию не установлен, оставляем пустым и ждем войти.
    
    7.1 Если по новому адресу ни чего не появилось в течении 10 минут, а на роутере горит "индикатор питания" (Оранжевый диод System) тогда поздравляю, вы что то сделали не так и закирпичили роутер, скорее всего подсунули не тот образ прошивки. В таком случае переходим к пункту раскирпичивания

# Раскирпичивание в случае неудачной прошивки (гайд от [stroti](https://4pda.to/forum/index.php?showuser=4540991))

Тут есть 2 пути, воспользоваться программой заботливо сделанной китайцами из Xiaomi или поднять свой tftp сервер, мы пойдем по пути наименьшего сопротивления и выберем первый вариант.

Оригинальная инструкция с картинками

1. Скачиваем [Xiaomi Recovery Tool](http://bigota.miwifi.com/xiaoqiang/tools/MIWIFIRepairTool.x86.zip) (если не качается скопируйте адрес ссылки и вставьте в адресную строку новой вкладки)

2. Выставляем настройки сетевого интерфейса на ПК: IP 192.168.31.100 маска 255.255.255.0 и жмем окей

3. Распаковываем и запускаем MIWIFIRepairTool.x86.exe
    
    3.1. На первой странице программы выбираем стоковую прошивку, подойдет и dev версия miwifi_rb03_firmware_stable_1.2.7.bin (проверял лично) далее жмем на 3 вопросика в правом нижнем углу, на след странице оставляем все как есть и жмем на самые правые ??? в правом нижнем углу

    3.2. Подходим к роутеру -> отсоединяем кабель питания -> зажимаем кнопку reset > вставляем кабель питания и не отпускаем кнопку reset, пока не заморгает желтый светодиод system
    
    3.3. После того как заморгает, отпускаем кнопку reset, а на экране компьютера с программой можно наблюдать полоску с процессом восстановления прошивки, дожидаемся окончания, на роутере светодиоды загорятся синими начнут моргать, заходим на роутер по адресу 192.168.31.1 (не забываем сбросить настройку сетевого интерфейса в винде)

    На этом кирпич восстановлен, и можно повторить путь прошивки с начала.

## Credits

- [Microsoft Azure](https://azure.microsoft.com)
- [GitHub Actions](https://github.com/features/actions)
- [OpenWrt](https://github.com/openwrt/openwrt)
- [Lean's OpenWrt](https://github.com/coolsnowwolf/lede)
- [tmate](https://github.com/tmate-io/tmate)
- [mxschmitt/action-tmate](https://github.com/mxschmitt/action-tmate)
- [csexton/debugger-action](https://github.com/csexton/debugger-action)
- [Cowtransfer](https://cowtransfer.com)
- [WeTransfer](https://wetransfer.com/)
- [Mikubill/transfer](https://github.com/Mikubill/transfer)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)
- [ActionsRML/delete-workflow-runs](https://github.com/ActionsRML/delete-workflow-runs)
- [dev-drprasad/delete-older-releases](https://github.com/dev-drprasad/delete-older-releases)
- [peter-evans/repository-dispatch](https://github.com/peter-evans/repository-dispatch)

## License

[MIT](https://github.com/P3TERX/Actions-OpenWrt/blob/main/LICENSE) © [**P3TERX**](https://p3terx.com)
