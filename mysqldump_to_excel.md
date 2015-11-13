# Исходный xml #

`mysqldump --xml -psomepassword databasename`

```
<?xml version="1.0"?>
<mysqldump xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<database name="databasename">

  <table_structure name="tasks">
    <field Field="user_id" Type="int(11)" Null="YES" Key="" Extra="" />
    <field Field="title" Type="varchar(20)" Null="YES" Key="" Extra="" />
    <options Name="tasks" Engine="MyISAM" ... />
  </table_structure>

  <table_data name="tasks">
    <row>
      <field name="user_id">1</field>
      <field name="title">tx</field>
    </row>
    ...
  </table_data>

  <table_structure name="users">
    <field Field="id" Type="int(11)" Null="NO" Key="PRI" Extra="auto_increment" />
    <field Field="name" Type="varchar(100)" Null="YES" Key="" Extra="" />
    <field Field="active" Type="tinyint(1)" Null="YES" Key="" Extra="" />
    <options ... />
  </table_structure>

  <table_data name="users">
    <row>
      <field name="id">1</field>
      <field name="name">Мегамозг</field>
      <field name="active">0</field>
    </row>
    ...
  </table_data>
</database>
</mysqldump>
```

# Формат Excel #

`apt-get install xalan`

`vim t.xsl`

```
<?xml version="1.0" encoding="utf-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
<xsl:output method="xml" encoding="utf-8"/>
<xsl:template match="/mysqldump/database/table_data">
  <xsl:if test="@name='users'">

    <Workbook
      xmlns="urn:schemas-microsoft-com:office:spreadsheet"
      xmlns:ss="urn:schemas-microsoft-com:office:spreadsheet">
    <Worksheet ss:Name="Пользователи">
    <Table>

    <Row>
      <Cell><Data ss:Type="String">№</Data></Cell>
      <Cell><Data ss:Type="String">Имя</Data></Cell>
      <Cell><Data ss:Type="String">Активен</Data></Cell>
    </Row>

    <xsl:for-each select="row">
    <Row>

      <xsl:for-each select="field">
      <Cell>
      <Data ss:Type="String">
      <xsl:choose>

        <xsl:when test="@name='active'">
        <xsl:choose>
          <xsl:when test=".=0">нет</xsl:when>
          <xsl:when test=".=1">да</xsl:when>
        </xsl:choose>
        </xsl:when>

        <xsl:otherwise>
          <xsl:value-of select="."/>
        </xsl:otherwise>

      </xsl:choose>
      </Data>
      </Cell>
      </xsl:for-each>

    </Row>
    </xsl:for-each>
    
    </Table>
    </Worksheet>
    </Workbook>
  </xsl:if>
</xsl:template>
</xsl:stylesheet>
```

`mysqldump --xml -psomepassword databasename | xalan -xsl t.xsl -out dump-$(date +"%H.%M-%d-%m-%Y").xml`