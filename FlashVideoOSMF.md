# Подготовка #

  * [Скачать](http://www.adobe.com/devnet/flex/flex-sdk-download.html) Flex SDK (содержит osmf)
  * [Скачать](http://www.flashdevelop.org/) Flash Develop (ide)

# Программирование #

  * Запустить Flash Develop
  * Создать проект: Project - New Project - ActionScript3 Flex 4 Project
  * Project - Properties:
    * Указать output file, dimension и т.д.
    * Выбрать Flex SDK: AS3 Context - Language - Installed Flex SDKs - ... - указать папку с sdk
    * Явно указать путь до osmf.swc для работы intellisense: Compiler Options - SWC Libraries - путь к swc-файлу
  * Основной as-файл:
    * Справа ПКМ на файл - Document Class (сделать главным)
    * Содержимое:
```
package
{
	import flash.display.Sprite;
	import org.osmf.media.MediaPlayerSprite;
	import org.osmf.media.URLResource;
		
	public class Test extends Sprite
	{
		private var mps:MediaPlayerSprite;
		public function Test()
		{
			mps = new MediaPlayerSprite();
			addChild(mps);
			mps.resource = new URLResource("echo.mp4");
		}
	}
}
```
  * Скомпилировать: Project - Build Project (или Test Project для запуска)