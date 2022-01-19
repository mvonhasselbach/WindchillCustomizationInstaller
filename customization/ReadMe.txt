General: 
1. use windchill shell for the commands mentioned below.
2. prerequisite is the installation of the installer by extracting installer.zip to <wt_home>

Install with the following command:
On Windows the simplest is to use the GUI:
type in a Windchill shell: 
	cd customization
	ant
The GUI will prompt you to select the Customization Package Zip File to install and to enter an WIndchill admin user and password and that's it!

You could as well use the command line version (especially on unix):
either:
	ant -f customization/build.xml install -Dfile=<filepath of customization zip file>
		e.g.: ant -f customization/build.xml install -Dfile="C:\Documents and Settings\mvonhasselbach\Desktop\federationBasics.zip"
		installs a customization package from the file 'federationBasics.zip' on my desktop
If you have loadFiles in your package, you have to append '-Ddbuser=<wtAdminUsr> -Dpasswd=<wtAdminPwd>' to the commands listed above.


Build basic package:
You can use the command as for installation and via the menue you can come to the 'Build package' screen.
You can as well use the command line version:
 with the following command:
	ant -f customization/build.xml buildPckg -DsinceDays=2 -Dname=federationBasics
		or
	ant -f customization/build.xml buildPckg -DsinceMinutes=200 -Dname=federationBasics

Build Package in the following way:
1. all custom files are below the base directory: <wt.home>/customization/<pckgName> = pckgDirRoot
2. there are multiple subfolders for different types of changes:
	- modify: for config files that are altered during the istall process, namely: 
			.xconf files, 
			.property files that are not controlled by xconf
			.xml files that get additions at specific locations like actions.xml and ActionsAndNavigations.xml
		the modification files must be at the same path relative to <pckgDirRoot> then the files to modify relative to <wt.home>
		e.g.: if you want to add entries to <wt.home>/codebase/wt/templateutil/ActionsAndNavigations.xml you have to 
		put these additions in <wt.home>/customization/<pckgName>/modify/codebase/wt/templateutil/ActionsAndNavigations.xml.
		The format of the files depends on their 'type':
		1. Files where the additions only get appended (.property) or inserted before the closing xml root node (.xconf, LogicalAttributes.xml) 
		the modif file just lists the content to append. 
		2. For xml file additions where the location in the modified doc is relevant you have to follow a specific format (see example) 
	- overwrite: for resource files (like .class, .java, .rbinfo, .ser, .xsl, IE tasks without task delegates) that just have to be copied to <wt.home>
	- addDB: for files that effect changes in the databases, like loadFiles, importable zip/jar files (oracle changes) 
		or .ptctar, .ptcdar and .ldif files (ldap changes).
		For loadFiles and importable zip/jar files: the folder of the files can be used to represent the container to load the data to:
		if the folderName contains an '=' sign it is assumed that it represents a container spec according the format used in the loadFromFile command,
		e.g.: the file with the path: 
		'<WT_HOME>\customization\installtest\addDB\wt.inf.container.OrgContainer=TST 1\wt.inf.containerLibrary=Engine 3.4 tdi\TimelineChart_csv.xml'
		and <packageName>=installtest means that the TimeLineChart_csv.xml will be loaded to the library named 'Engine 3.4 tdi' in the Organization named 'TST 1'
		Required content (e.g. report qml files) will be copied to the Windchill loadFile subdirectories, so that the paths in the csvxml loadfiles can be resolved properly.
	- scripts: to be as flexible as possible you can put ant tasks here that are executed just after the extraction of the package. 
		These ant tasks inherit the properties and refs from the installer, especially ${myPckgName}. The basedir is the wt_home/customization dir!
	- doc: the content will be copied to <wt_home>/customization/doc.
	- components: To support modularization of customizations you can add install kit zip files to this first-level folder. 
	  These sub-components will always be installed before the embracing installation kit. If you need to control the sequence of the installation of multiple included components 
	  you can do so by including a file /components/components.properties that can have the following entries:
		- sequence: value is a comma-separated list of component install kit filenames in the sequence they should be installed
		- <install kit filename>.requiresRestart: value is ‘true’ (or any other).  This controls if a restart is forced for a sub-component installation 
		  (depends on the content of the sub-component; e.g. if an addition to the State EnumType is followed by loading an LCTemplate that uses this 
		  added LC State). Only add a property if a restart is required as the installer just checks if there is a property and not its value!

		
	in general: for all files that are modified a backup of the original file is done in one zip file per installation process.
	this zip file is named: <tstamp>_<pckgName>_rollback.zip and is put to <wt.home>/customization/rollback and can be used to rollback the installation
	(not rolling back db changes!).  
	Also a log is created for every install run and is added to the rollback.zip .
	
ChangeLog:
	08/14/2012: a lot happened since 2007: components support, export jar/zip file loading, automatic restart, a wizard UI, fix of Unix bugs
	03/20/2007: added documentation about Unix installation issue due to bug in ant's ootb unzip functionality on unix
	11/29/2006: added support for wtSafeArea: all modified files are copied to the corresponding subdirs of wtSafeArea 
	08/24/2006: added support for replacement of xml file additions by 'identifier' attribute. installer is backward compatible to packages w/o this attribute in its configs.
					added documentation for 'identifier' attribute in this ReadMe.txt.
	02/15/2006: added support for ant script execution in scripts directory
					catched exceptions when one of the root dirs does not exist -> more robust
	12/02/2005: added import of .ldif files
	09/06/2005: not yet implemented:
		- import of IXB zip/jar files (e.g.: wf/lc export files): as a workaround you should simply extract these zip/jar files to <pckgDirRoot>/addDB
		- import of .ldif files: I couldn't find the code that they use in the I*E Administrator :(
		- no verify functionality for file modifications e.g: only add fragment if not yet existant, otherwise modify
		
		
Example of file that specifies additions to xml files at a specific location:

<?xml version="1.0"?>
<modif>
	<fragment 	identifier="/tabmodels/model[@name='user navigation']/action[@name='list_mvh'][@type='work']" 
					path="/tabmodels/model[@name='user navigation']/action[@name='list'][@type='work']" position="after"><![CDATA[
                <action name="list_mvh"               type="work"/>
	]]>
	</fragment>
	<fragment 	identifier="/tabmodels/model[@action='list_mvh'][@type='work']" 
					path="/tabmodels/model[@action='list'][@type='work']" position="after"><![CDATA[
        <model type="work" action="list_mvh">
                <connect name="user navigation" selectedName="list_mvh" selectedType="work"/>
        </model>
	]]>
	</fragment>        
</modif>


This is an addition to nav.xml:
<modif> is the root node
<fragment> specifies where to insert. It's 'identifier' attribute specifies the fragm,ent to insert for existance checking and replacement if required, the format is an XPath expression.
	The 'path' attribute selects the node relative to which your fragment is inserted, the format is an XPath expression.
	The 'position' specifies where to insert it, values are 'after','before' and 'under'.
	The content (nonrmally inside the <![CDATA[ ]]> section or encoded otherwise) specifies what to add .

you can have as many <fragment> elements as you want!

Here are examples for all commonly used files:
actions.xml:
	<modif>
		<fragment identifier="/listofactions/objecttype[@name='object']/action[@name='clipboardContent']" 
						path="/listofactions/objecttype[@name='object']/action[@name='clearClipboard']" position="after"><![CDATA[
	      <action name="clipboardContent" renderType="GENERAL" by="mvh">
	         <command method="/wtcore/jsp/ext/utils/clipboardContent.jsp"  windowType="popup"/>
	      </action>
		]]>
		</fragment>
	</modif>
	
actionmodels.xml:
	<modif>
		<fragment identifier="/actionmodels/model[@name='project context info bar actions']/action[name='clipboardContent']" 
						path="/actionmodels/model[@name='project context info bar actions']/action[position()=last()]" position="after"><![CDATA[
	      <action name="clipboardContent"     type="object" 	by="mvh"/>
			]]>
		</fragment>
		<fragment identifier="/actionmodels/model[@name='project context info bar actions']/action[name='clipboardContent']" 
						path="/actionmodels/model[@name='product context info bar actions']/action[position()=last()]" position="after"><![CDATA[
	      <action name="clipboardContent"     type="object" 	by="mvh"/>
			]]>
		</fragment>
	</modif>
	
nav.xml: (see explained example)

config/mvc/custom.xml: in this one we have to deal with namespaces - default (see ':' in XPaths and xmlns declaration in <beans/> addition) 
and explicit (see xmlns:context declaration in <context:component-scan/> addition). This is due to the namespace declarations in the target custom.xml file.
	<modif>
		<fragment identifier="/:beans/:bean[@class=’ext.part.mvc.PSBTreeConfigBuilder’]" 
		          path="/:beans" position="under"><![CDATA[
			<bean xmlns="http://www.springframework.org/schema/beans" class="ext.part.mvc.PSBTreeConfigBuilder" />
			]]>
		</fragment>
		<fragment identifier="/:beans/context:component-scan[@base-package='ext.part.mvc']" 
		          path="/:beans" position="under"><![CDATA[
		<context:component-scan xmlns:context="http://www.springframework.org/schema/context" base-package="ext.part.mvc" use-default-filters="true">
		   <context:include-filter xmlns:context="http://www.springframework.org/schema/context" type="annotation" expression="com.ptc.cat.entity.server.MappedAttributeDataTypeOverride"/>
		</context:component-scan>
		]]>
		</fragment>
	</modif>

NavigationAndActions.xml:
	<modif>
		<fragment identifier="/NAVIGATION_TREES/NAVIGATION_TREE[Name='WTDOCUMENT_DOCUMENTSTRUCTURE_TABLE_ACTIONS']/Links/Link[Name='PASTETOUSES']" 
						path="/NAVIGATION_TREES/NAVIGATION_TREE[Name='WTDOCUMENT_DOCUMENTSTRUCTURE_TABLE_ACTIONS']/Links/Link[Name='DocStrctRemoveChildren']" position="after"><![CDATA[
	        <!--mvh start: add4DocMgmt paste-->
	            <Link>
	                <Name>PASTETOUSES</Name>
	                <Action>PASTETOUSES</Action>
	                <Tree_Action_Delegate/>
	                <In_Minimum_List>true</In_Minimum_List>
	                <Resource_Bundle>com.ptc.core.HTMLtemplateutil.server.processors.processorsResource</Resource_Bundle>
	                <Resource_Key>4</Resource_Key>
	                <Links/>
	            </Link>
	        <!--mvh end: add4DocMgmt paste-->
		]]>
	 	</fragment>
		<fragment identifier="/NAVIGATION_TREES/NAVIGATION_TREE[Name='DocumentReferencedBy_TableActions']" 
						path="/NAVIGATION_TREES/NAVIGATION_TREE[Name='DocumentReferences_TableActions']" position="after"><![CDATA[
	     <!--mvh start: add4DocMgmt paste-->
	    <NAVIGATION_TREE>
	        <Name>DocumentReferencedBy_TableActions</Name>
	        <Links>
	            <Link>
	                <Name>PASTETOREFERENCEDBY</Name>
	                <Action>PASTETOREFERENCEDBY</Action>
	                <Tree_Action_Delegate/>
	                <In_Minimum_List>true</In_Minimum_List>
	                <Resource_Bundle>com.ptc.core.HTMLtemplateutil.server.processors.processorsResource</Resource_Bundle>
	                <Resource_Key>4</Resource_Key>
	                <Links/>
	            </Link>
	        </Links>
	    </NAVIGATION_TREE>
		]]>
	 	</fragment>	     
	</modif>
    