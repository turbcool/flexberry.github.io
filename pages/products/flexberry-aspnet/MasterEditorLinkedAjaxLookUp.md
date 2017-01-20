---
title: MasterEditorLinkedAjaxLookUp
sidebar: product--sidebar
keywords: Flexberry ASP-NET
toc: true
permalink: ru/master-editor-linked-ajax-look-up.html
folder: product--folder
lang: ru
---

# Описание
Данный контрол позволяет редактировать и свойство и мастера в одной ячейке [AjaxGroupEdit](ajax-group-edit.html). 

(((<msg type=Warning>Данная статья описывает создание `MasterEditorLinkedLinkedLookup`, расположенных в AGE, а не непосредственно на форме редактирования! 

На формах редактирования необходимо использовать логику [Связывание AjaxAutocomplete и AjaxLookup](link--ajax-autocomplete-and--ajax-lookup.html).</msg>)))

# Постановка задачи
Предположим, у нас есть диаграмма классов:

[imageauto||http://wiki.ics.perm.ru/GetFile.aspx?File=LinkedLookupDiagramm.png&AsStreamAttachment=1&Provider=ScrewTurn.Wiki.Plugins.SqlServer.SqlServerFilesStorageProvider&IsPageAttachment=1&Page=MasterEditorLinkedAjaxLookUp&NoHit=1]

Необходимо при добавлении `СтрокиПланаПогашения` дать возможность
* либо выбрать `БанковскуюКарту` из списка, 
* либо вбить `НомерКарты` вручную, и 
** при совпадении с существующим номером проставить ссылку на `БанковскуюКарту` автоматически,
** а при несовпадении просто сохранить введенную пользователем информацию в поле `НомерКарты`.

# Подключение и настройка
Алгоритм подключения и настройки `LinkedLookup`:
* Настроить представление детейла.
* Добавить настройку в [WebControlProvider](web-control-provider.html).
* Добавить настройку в [WebBinder](web-binder.html).
* Настроить LookUp при помощи [LookUpSettings](http://storm:20013/class_i_c_s_soft_1_1_s_t_o_r_m_n_e_t_1_1_web_1_1_controls_1_1_web_group_edit_1_1_look_up_setting.html)

## Настройка представления детейла
В [E-представлении](e-view.html) детейла необходимо добавить все необходимые поля (обязательно добавить `БанковскуюКарту`, `БанковскаяКарта.Номер` и `НомерКарты`), но при этом снять видимость с собственного поля, в котором будет храниться информация (в нашем случае это `НомерКарты`).

## Настройка WebControlProvider
В WebControlProvider необходимо добавить информацию о том, что для редактирования ссылки `СтрокаПланаПогашения` - `БанковскаяКарта` должен использоваться контрол `MasterEditorLinkedAjaxLookUp`. Делается это следующим образом:

```xml
  <customproperty class="СтрокаПланаПогашения" property="Карта">
    <control typename="System.Web.UI.WebControls.Label, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" property="Text" codefile="" />
    <editcontrol typename="ICSSoft.STORMNET.Web.AjaxControls.MasterEditorLinkedAjaxLookUp" codefile="" />
  </customproperty>
  <customproperty class="СтрокаПланаПогашения" property="НомерКарты">
    <control typename="System.Web.UI.WebControls.Label, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" property="Text" codefile="" />
    <editcontrol typename="ICSSoft.STORMNET.Web.AjaxControls.MasterEditorLinkedAjaxLookUp" codefile="" />
  </customproperty>
```

## Настройка биндинга
Для того, чтобы контрол корректно работал для него нужно настроить нестандартный биндинг (при помощи [WebBinder](web-binder.html)).

Для этого:

* Необходимо в папке '''~/XML/Bindings/''' создать XML-файл с названием, совпадающим с названием формы редактирования, на которой будет располагаться детейл (то есть на форме редактирования агрегатора). В нашем случае он будет называться '''KreditE.xml'''.
* В этот файл необходимо добавить нестандартный биндинг, который будет сообщать системе, что один контрол должен редактировать сразу два свойства.

В нашем случае это будет выглядеть следующим образом:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<root partial="true">
  <detail name="СтрокаПланаПогашения">
    <property name="Карта">
      <control id="ctrlКарта" prop="SelectedMasterPK">
      </control>
    </property>
    <property name="НомерКарты">
      <control id="ctrlКарта" prop="Text">
      </control>
    </property>
  </detail>
</root>
```

(((<msg type=Important>Очень важно следить за правильностью составления этого биндинга:
* Указать, что биндинг составной: добавить блок `partial="true"` в тэг `root`.
* Указать имя детейла, для которого составляется этот биндинг (в нашем случае `СтрокаПланаПогашения`).
* Указать наименования двух свойств, которые должны редактироваться (в нашем случае `Карта` и `НомерКарты`).
* Указать ID контрола. Т.к. контрол находится в [AjaxGroupEdit'e](ajax-group-edit.html), то в явном виде его наименования на странице найти невозможно. Имя формируется как `ctrl` + имя ссылки на мастера (в нашем случае `ctrl` + `Карта`), __обязательно следите за регистром, `ctrlКарта` это не то же самое, что `ctrlкарта`__!
* Наименование редактируемых свойств (`SelectedMasterPK` и `Text` всегда остаются неизменными, но важно понимать, что `Text` относится к свойству детейла, а `SelectedMasterPK` к мастеру детейла.</msg>)))


## Настройка лукапа
Чтобы при вводе текста система подсказывала пользователю существующие варианты, необходимо настроить лукап, указав ему `LookUpSetting`:

```

protected override void Preload()
{
    ctrlСтрокаПланаПогашения.AddLookUpSettings(
        Information.ExtractPropertyPath<СтрокаПланаПогашения>(x => x.Карта), new LookUpSetting
        {
            LookUpBtnVisible = true,
            LookUpClearBtnVisible = true,
            Autocomplete = true,
            IsSubstring = true
        });
}
```

(((<msg type=note>Хочется еще раз напомнить, что все настройки осуществляются на форме редактирования класса-агрегатора, то есть на форме `KreditE`.</msg>)))


# Возможные проблемы
Если в базе уже есть данные, то при подключении `MasterEditorLinkedAjaxLookUp` могут возникать проблемы при отображении "старых" данных.
