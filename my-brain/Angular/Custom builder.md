### 🧠 **Зачем писать свой Angular builder?**

Angular CLI использует систему builders для различных задач: сборка (`build`), запуск (`serve`), тесты (`test`), линтинг (`lint`), и т.д. Если тебе не хватает возможностей стандартных builders или ты хочешь автоматизировать/изменить процесс под себя — тогда имеет смысл создать кастомный.

---

### 💡 **Что может дать кастомный builder:**

1. **Оптимизация сборки под свой проект.**  
    Например, автоматическое удаление ненужных зависимостей, генерация специфических файлов, дополнительная минификация и т.д.
    
2. **Интеграция с другими системами.**  
    Например, перед билдом дернуть API и получить конфиг, или залить артефакты после сборки.
    
3. **Автоматизация рутинных задач.**  
    Например, автоматическое изменение `environment` файлов, управление переменными окружения, вставка даты сборки и т.д.
    
4. **Расширение возможностей стандартной сборки.**  
    Например, добавить специфическую обработку ассетов или свой формат трансформации CSS/JS.
    

---

### ✅ **Пример: кастомный Angular builder, который вставляет дату сборки**

Допустим, ты хочешь, чтобы в `environment.ts` автоматически добавлялась метка времени сборки:

ts

КопироватьРедактировать

`// environment.ts export const environment = {   production: true,   buildTimestamp: '__BUILD_TIMESTAMP__' };`

Создаёшь кастомный builder, который на лету заменяет `__BUILD_TIMESTAMP__` на текущую дату перед сборкой.

---

### 🛠️ **Как это реализовать (вкратце):**

1. Создаёшь npm-пакет со своим builder-ом.
    
2. Реализуешь функцию `builder` с использованием API из `@angular-devkit/architect`.
    
3. В `angular.json` указываешь свой builder вместо стандартного.
    

`"architect": {   "build": {     "builder": "@my-org/builders:custom-build",     "options": { ... }   } }`

---

### 📦 Пример кода builder-а:

ts

КопироватьРедактировать

``` js
import { BuilderContext, BuilderOutput, createBuilder } from '@angular-devkit/architect'; 
import { from } from 'rxjs'; import * as fs from 'fs';  

function buildWithTimestamp(options: any, context: BuilderContext): any {   const filePath = 'src/environments/environment.prod.ts';   
const content = fs.readFileSync(filePath, 'utf-8');   
const newContent = content.replace('__BUILD_TIMESTAMP__', new Date().toISOString());   

fs.writeFileSync(filePath, newContent);    

context.logger.info('🔨 Updated build timestamp!');    
// Затем вызываем стандартный angular build   
return from(     
context.scheduleTarget({ target: 'build', project: context.target?.project! }).then(target => target.result)   ); 
}  

export default createBuilder(buildWithTimestamp);
```
