# Аккорды

Данный модуль позволяет вам задавать аккорды любой длины, причём в отличие от стандартной фичи [combos](https://docs.qmk.fm/#/feature_combo) в QMK, данный модуль способен посылать любые кейкоды. Причём данные аккорды способны работать не только в качестве "нажал-отпустил", а качестве зажимаемых клавиш, будь то переключение слоя или модификатор.

Аккорд срабатывает в следующих случаях:
* Если вы зажимаете клавиши для аккорда более 100 миллисекунд, то считается что вы больше не нажмёте клавиш для следующего аккорда, и кейкод текущего аккорда нажимается. Например, можно задать на клавише `CMB_001` аккорд из одной этой клавиши, который при срабатанывании будет нажимать `KC_BSPC` (Backspace), таким образом если вы зажмёте клавишу `CMB_001`, то у вас сначала будет ожидание 100мс, затем нажмётся клавиша Backspace, и начнёт стираться текст. То есть клавиши для аккордов можно свободно использовать даже для клавиш, которые предполагается зажимать, а не только нажать и отпускать.
* Если вы во время зажатия аккорда нажимаете другую клавишу, то считается что вы больше не нажмёте клавиш для следующего аккорда, и кейкод текущего аккорда нажимается.

Если вы хотите более подробно понять как работает обработка аккордов, или хотите модифицировать код, смотрите секцию [принцип работы](#принцип-работы).

Советую вам не бояться нажимать две клавиши одним пальцем, и помещать на такие нажатия аккорды.

# Как использовать?

Приготовьтесь, использовать довольно сложно, потому что данный модуль построен на костылях и хаках (если придумаете как сделать иначе - делайте PR!).

Последовательность действий будет описана так, чтобы вы модифицировали свой файл `keymap.c` сверху-вниз.

## Подключить другой модуль

Необходимо подключить и настроить модуль [arbitrary_keycode](https://github.com/klavarog/arbitrary_keycode). Подключение файла из этого модуля необходимо поместить раньше подключения текущего модуля.

## Задать характеристики

В `config.h` нужно прописать задание следующих переменных, модифицируя их под свои нужды:

* `#define COMBO_KEYS_COUNT 5` - количество используемых клавиш для аккордов, для данной опции у вас в итоге получатся клавиши `CMB_000`, `CMB_001`, ... , `CMB_004`.
* `#define COMBO_MAX_SIZE 3` - максимальное количество одновременно зажимаемых клавиш для одного аккорда, больше этого размера аккорд задавать нельзя. При больших значениях потребляет больше памяти.
* `#define COMBO_STACK_MAX_SIZE 3` - максимальное количество одновременно зажимаемых аккордов. То есть, например, у вас есть аккорд для получения шифта, вы его зажимаете, затем вы нажимаете другой аккорд один раз, это значит что максимально у вас было 2 одновременно зажатых аккорда. 3 должно хватить для всех целей.
* `#define COMBO_WAIT_TIME 100` - время в миллисекундах в течении которого ждётся что все клавиши текущего аккорда будут нажаты. Если это время истекло, и текущую комбинацию нажатых клавиш можно трактовать как аккорд, то именно эта комбинация и пошлётся.

## Задание SAFE_RANGE

Для пользовательских кейкодов существует такое понятие как `SAFE_RANGE`, обычно он используется для того чтобы после него помещать свои кейкоды, которые делают особые действия:

```c
enum custom_keycodes {
  MY_KEY1 = SAFE_RANGE,
  MY_KEY2,
  // ...
}
```

Таким образом гарантируется что пользовательские кейкоды не пересекутся с системными кейкодами. Обычно для этого используется `SAFE_RANGE`.

Но иногда случается так, что клавиатура добавляет своих кейкодов, и создаёт новую переменную SAFE_RANGE, которую называет по-другому, нужно за этим следить. Посмотрите, нету ли у вас такой особой переменной. Например, в Moonlander эта переменная называется `ML_SAFE_RANGE`.

После того как вы найдёте имя этой переменной, в keymap.c прямо перед включением `combo` нужно написать:

```c
#define CUSTOM_SAFE_RANGE <ваша переменная>
```

Пример для Moonlander:
```c
#define CUSTOM_SAFE_RANGE ML_SAFE_RANGE
```

Или если вы уже используете `lang_shift`, то это делать не нужно, `lang_shift` yже переопределяет эту переменную. То есть должно получиться что-то вида:

```c
#define CUSTOM_SAFE_RANGE ML_SAFE_RANGE
#include "lang_shift/include.h"
#include "combo/include.h"
```

Затем при использовании своих кейкодов необходимо указывать именно `CUSTOM_SAFE_RANGE`, потому что данный модуль использует необходимое количество кейкодов из пользовательских кейкодов и переопределяет эту переменную. Пример:

```c
enum custom_keycodes {
  KEYCODES_START = CUSTOM_SAFE_RANGE,

  // English specific keys
  EN_LTEQ, // <=
  // ...
};
```

## Подключение кода

В своём файле `keymap.c` в самом верху подключаем файл `combo/include.h`:
```c
#include "combo/include.h"
```

## Записать аккорды

Далее надо записать аккорды и зажимаемые для них клавиши. Записывается при помощи макроса `CHORD`, где первым аргументом передаётся кейкод, который будет нажат (там можно задать даже кастомный кейкод, и переключение слоя), затем нужно указать клавиши `CMB_***` одновременное зажатие которых будет посылать зажатие данного аккорда.

```c
const ComboWithKeycode combos[] PROGMEM = {
  // Left Index
  CHORD(MO(4),         CMB_000),
  CHORD(MY_KEY,        CMB_001),
  CHORD(LGUI(S(KC_A)), CMB_000, CMB_001),
  // ...
 };
const uint8_t combos_size = sizeof(combos)/sizeof(ComboWithKeycode);
```

Здесь при единичном зажатии клавиши `CMB_000` будет включаться 4 слой, а при нажатии этой клавиши одновременно с `CMB_001` будет нажиматься `Win+Shift+A`.

## Моментальный аккорд

В данной библиотеке есть такая фича как **моментальный аккорд**. Его суть заключается в том, что при нажатии клавиш, которые его образуют, он нажимается моментально, без ожидания `COMBO_WAIT_TIME`. Если же через время меньше, чем `COMBO_WAIT_TIME`, нажимается другая клавиша, образующая текущий аккорд, то нажатая клавиша **отменяется**, и продолжается формирование другого аккорда. Если же проходит заданное время, или нажимается другая, неаккордовая клавиша, то считается что данный аккорд случился, и он больше не отменится.

Задаётся моментальный аккорд через макрос `IMMEDIATE_CHORD(KEYCODE, UNDO_KEYCODE, ...)`. Где при обычном нажатии нажимается `KEYCODE`, а в случае отмены _отпускается_ кейкод, заданный в `UNDO_KEYCODE`. Обычно `UNDO_KEYCODE` нужно писать такой же, как и `KEYCODE`, но бывают случаи когда они могут различаться. Например, если в `KEYCODE` стоит залипающий шифт, то нельзя задать там одинаковые кейкоды, ведь нажатие и отжатие залипающего шифта может привести к тому, что он применится к следующей клавише. А если залипающий шифт является составной частью других аккордов, то получится так, что при нажатии сложного аккорда, заключащего в себе залипающий шифт, может моментально активироваться залипающий шифт и применится к следующей клавише. Поэтому отжатие кейкода и отмена различаются. Поэтому для отмены залипающего шифта вы можете написать особый обработчик.

Или, например, в случае, если моментальный аккорд стоит на букве, то отмена буквы будет являться её отжатие и посылка <kbd>Backspace</kbd>.

Моментальный аккорд может пригодиться в следующем случае: если клавиша <kbd>Shift</kbd> стоит на аккорде, то хочется чтобы она нажималась сразу. Это актуально при работе с визуальными редакторами, где параллельно происходит работа с мышью. И даже задержки отправки <kbd>Shift</kbd> в 100мс тут могут оказать значительное неудобство, поэтому <kbd>Shift</kbd> необходимо делать моментальным аккордом, а его единоразовое нажатие в случае, если он является составной частью других аккордов, ничего не стоит.

В иных случаях моментальный аккорд особо не нужен, потому что работа с аккордами и другими клавишами в данном расширении реализована хорошо, и вас не должны волновать тайминги.

Не используйте моментальный аккорд для клавиш типо <kbd>Backspace</kbd>, которую невозможно отменить клавиатурными нажатиями. Иначе в составе других аккордов она будет случайно нажиматься и будет сильно мешать. Моментальный аккорд рекомендуется использовать только для слоефикаторов и модификаторов.

## Поместить аккорды на вашу раскладку

Используйте кейкоды `CMB_000`-`CMB_XXX` (зависит сколько вы задали их в начале файла).

```c
const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {
    //---------------------------------------------------------------------------
  [0] = MY_layout(
    // ...
    XXXXXXX,    XXXXXXX,  XXXXXXX,  CMB_003,  CMB_004,
    CMB_002,
    CMB_000,    CMB_001,  KC_ENT,
    // ...
  ),
  // ...
```

## Вызов модуля для каждой клавиши

В функции `process_record_user` в самом начале, перед всеми проверками, добавляем код обработки аккорда:
```c
bool process_record_user(uint16_t key, keyrecord_t *record) {
  if (!combo_process_record(key, record)) return false;

  // ...
}
```

Если вы используете `lang_shift`, то обработка аккордов должна располагаться **перед** обработкой `lang_shift`! То есть:

```c
bool process_record_user(uint16_t key, keyrecord_t *record) {
  if (!combo_process_record(key, record)) return false;
  if (!lang_shift_process_record(keycode, record)(key, record)) return false;

  // ...
}
```

Затем нужно определить функцию `matrix_scan_user`, если она у вас ещё не определена, и вызывать функцию `user_timer`, а в этой функции вызывать `combo_user_timer();`:
```c
void user_timer(void) {
  combo_user_timer();
}

void matrix_scan_user(void) {
  user_timer();
}
```

**Объяснение:** функция `matrix_scan_user` вызывается примерно каждые 2 миллисекунды, она сканирует матрицу. Значит её вполне можно использовать для отслеживания собственных таймеров. Поэтому мы вызываем из неё функцию `user_timer`, которая лучше говорит о наших намерениях, чем `matrx_scan_user`. А уже в функции `user_timer` мы вызываем обработку случая когда мы слишком долго держим аккорд.

Далее нужно определить функции для обработки ошибок:
```c
void combo_max_count_error(void) {
  // ...
}

void combo_max_size_error(void) {
  // ...
}

```

Эти функции будут вызываться в случае превышения статических размеров массивов, которые хранят текущие состояния аккордов. `combo_max_count_error` будет вызвано, когда у вас одновременно нажато больше аккордов, чем `COMBO_STACK_MAX_SIZE`. `combo_max_size_error` будет вызвано, когда у вас в одном аккорде нажато больше клавиш, чем задано в `COMBO_MAX_SIZE`. Рекомендуется на эти функции проигрывать особые звуки, либо мигать подсветкой/светодиодами, либо выводить принт через дебаг, либо посылать макрос на нажатие. Если какая-то из таких ошибок произошла, это означает, что соответствующую переменную максимального количества лучше увеличить.

# Принцип работы

![](dka.png)

Данная картинка нарисована через [draw.io](https://app.diagrams.net), хранится в файле `dka.drawio`, и чтобы получить данную картинку надо указать следующие настройки при экспорте как PNG: `Zoom: 150%`.

Здесь показано как работает обработка аккордов. Оранжевые прямоугольники показывают условиях перехода; белые показывают ждущие состояния; зелёные показывают действия. В белых прямоугольниках подписан номер состояния, которое хранится в `combo->state`, а в оранжевых подписана буква перехода, которую можно найти в коде по использованию макроса `TRANSITION_DEBUG()`.

Данный конечный автомат показывает обработку для одного аккорда, находящегося в стэке. Порядок обработки следующий:
* Сначала обрабатываем все аккорды в стэке согласно этому ДКА, если хотя бы один сработал, завершаем всю обработку.
* Если ни один не сработал, то обрабатываем текущую клавишу синим прямоугольником.

Таким образом мы можем добавлять новые аккорды в стэк если они не добавляют ничего к уже зажатым, и можем добавлять к зажатым ещё клавиш, если в итоге можно что-то нажать.

# Changelog

**23.01.2021**:
* Добавлен моментальный аккорд.

**20.01.2021**:
* Теперь надо писать `PROGMEM` у массива `combos`.
* Теперь надо определить функции для логирования ошибок: `combo_max_size_error`, `combo_max_count_error`.