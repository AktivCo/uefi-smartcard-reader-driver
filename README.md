# Описание проекта

Этот проект -- форк [opensource-реализации](https://github.com/LudovicRousseau/edk2/tree/SmartCard) UEFI Smart Card Reader Protocol, выполненной [LudovicRousseau](https://github.com/LudovicRousseau/). [UEFI Smart Card Reader Protocol](https://www.uefi.org/sites/default/files/resources/UEFI%202_5.pdf#G39.1357371) был представлен в [спецификации UEFI версии 2.5](https://www.uefi.org/sites/default/files/resources/UEFI%202_5.pdf). В рамках форка были исправлены мелкие ошибки, не позволявшие собирать проект без предупреждений компилятора с актуальной версией [EDK II](https://github.com/tianocore/edk2). Исходный код драйвера выделен в отдельный пакет для упрощения ознакомления.

Реализация драйвера не проходила тщательного аудита кода и [не была принята в кодовую базу проекта edk2](https://www.mail-archive.com/edk2-devel@lists.sourceforge.net/msg14937.html), в основном, из-за неготовности мейнтейнеров проекта edk2 вносить в проект компоненты, лицензированные под LGPL. Эта версия драйвера может быть использована для запуска UEFI-модулей, взаимодействующих со смарт-картами, в UEFI-средах, где отсутствует встроенная реализация UEFI Smart Card Reader Protocol. После аудита кода и при условии соблюдения лицензии LGPL, в соответствии с которой распространяется драйвер, он может быть использован в UEFI-модулях, решающих прикладные задачи.

Лицензионные обязательства, которые необходимо учитывать при использовании и распространении ПО данного проекта, указаны в файле [SmartCardReaderPkg/SmartCardReader/License.txt](SmartCardReaderPkg/SmartCardReader/License.txt).

## Конфигурация и сборка

В репозитории проекта используются сабмодули. Перед сборкой, пожалуйста, убедитесь, что исходный код проекта скачан полностью. Для скачивания сабмодулей достаточно выполнить следующую команду из корневой директории проекта:

```
git submodule update --init --recursive
```

Для сборки проекта используется окружение [EDK II](https://github.com/tianocore/edk2). Для сборки проекта должны быть выполнены [предусловия](https://github.com/tianocore/tianocore.github.io/wiki/Getting-Started-with-EDK-II), включающие в себя установку инструментов для сборки. В частности, для сборки потребуется компилятор (GCC или Clang), Python, NASM.

### Сборка в Linux (gcc)

Для настройки среды сборки, воспользуйтесь инструкциями по [ссылке](https://github.com/tianocore/tianocore.github.io/wiki/Using-EDK-II-with-Native-GCC).

Для операционных систем, основанных на Debian, ожидается, что данная команда установит все необходиные инструменты:

```
sudo apt-get install build-essential uuid-dev iasl gcc-5 nasm python3-distutils
```

Для сборки используется компилятор gcc. Тестирование сборки производилось компилятором gcc 4.8.

Каждый шаг сборки выполняется из корневой директории проекта.

1. Выполнить сборку `BaseTools` проекта edk2:

    ```
    pushd edk2
    make -C BaseTools
    popd
    ```

2. Настроить `BaseTools` и переменные окружения:
    
    ```
    export WORKSPACE=$(pwd)
    export PACKAGES_PATH="$(pwd)/edk2:$(pwd)"
    ./edk2/edksetup.sh
    ```

3. Выполнить сборку UEFI-модуля примера:

    ```
    build -t GCC5 -a X64 -b RELEASE -p ./SmartCardReaderPkg/SmartCardReaderPkg.dsc
    ```

Исполняемый модуль примера доступен по пути `${WORKSPACE}/Build/SmartCardReaderPkg/RELEASE_GCC5/X64/SmartCardReader.efi`.

### Сборка в Windows (clang)

Перед сборкой требуется установить инструменты:
1) [Python](https://www.python.org/) версии 3.7 или выше
2) NASM - [инструкция по настройке](https://github.com/tianocore/tianocore.github.io/wiki/Nasm-Setup)
3) ASL - [инструкция по настройке](https://github.com/tianocore/tianocore.github.io/wiki/Asl-Setup)
4) [Clang](https://releases.llvm.org/download.html).

Для сборки используется компилятор clang. Тестирование сборки производилось компиляторами clang 9 и clang 11.

Каждый шаг сборки выполняется в командной строке из корневой директории проекта.

1. Настроить переменные окружения:
    ```
    set NASM_BIN=C:\Path\To\Your\NASMDirectory\
    set CLANG_BIN=C:\Path\To\Your\ClangDirectory\
    set PYTHON_HOME=C:\Path\To\Your\PythonRootDirectory\
    set WORKSPACE=%CD%
    set PACKAGES_PATH=%CD%\edk2;%CD%
    ```

2. Выполнить сборку `BaseTools` проекта edk2:

    ```
    .\edk2\edksetup.bat Rebuild 
    ```

3. Выполнить сборку UEFI-модуля примера:

    ```
    build -t CLANGPDB -a X64 -b RELEASE -p ./SmartCardReaderPkg/SmartCardReaderPkg.dsc
    ```

Исполняемый модуль примера доступен по пути `${WORKSPACE}/Build/SmartCardReaderPkg/RELEASE_CLANGPDB/X64/SmartCardReader.efi`.

## Использование

Для модулей, решающих прикладные задачи, стандартным методом внедрения в процесс загрузки компьютера является модификация глобальных NVRAM-переменных (см. [3 Boot Manager Unified Extensible Firmware Interface Specification 2.6](https://www.uefi.org/sites/default/files/resources/UEFI%20Spec%202_6.pdf#G7.1001195)). В частности, для загрузки драйвера требуется создание переменной Driver#### и внесение имени этой переменной в список загрузки, заданного в переменной DriverOrder.

Пример использования драйвера представлен в репозитории [Rutoken UEFI APDU Samples](https://github.com/AktivCo/rutoken-uefi-apdu-samples).
