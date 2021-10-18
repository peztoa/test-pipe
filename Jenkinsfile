/**
* Resumen.
* Objeto: Jenkinsfile
* Descripcion: El objeto Jenkinsfile contiene el script de pipeline para
* el despliegue de WARI mediante Jenkins
* Autor: HRL
*/
node{
    properties(
        [
            parameters([
                string(name: 'RUTA_DE_REPO_DE_PARCHE', defaultValue: params.RUTA_DE_REPO_DE_PARCHE, description: 'La URL del repositorio de parches, por favor asegurese que apunte al repositorio correcto', trim: false),
                string(name: 'ID_DE_CREDENCIALES_DE_REPO_DE_PARCHE', defaultValue: params.ID_DE_CREDENCIALES_DE_REPO_DE_PARCHE, description: 'El id de los credenciales configurados en jenkins para acceder al repositorio de parches, verifique que apunte a un id de credenciales existente', trim: false),
				string(name: 'BRANCH_DEL_REPO_DE_PARCHE_A_USAR', defaultValue: params.BRANCH_DEL_REPO_DE_PARCHE_A_USAR, description: 'Rama del repositorio de parches que contiene el archivo de propiedades para realizar el despliegue de este pipeline', trim: false),
				string(name: 'RUTA_DEL_ARCHIVO_DE_CONFIGURACION_EN_EL_REPO_DE_PARCHE', defaultValue: params.RUTA_DEL_ARCHIVO_DE_CONFIGURACION_EN_EL_REPO_DE_PARCHE, description: 'Ruta del archivo que contiene las propiedades para realizar el despliegue de este pipeline', trim: false),
                text(name: 'NOMBRES_DE_APLICACIONES_A_DESPLEGAR', defaultValue:  '''wari-migration-app-ear
wari-migration-app-reports-ear''', description: 'Los nombres de las aplicaciones a desplegar, cada renglon tendra una aplicacion a desplegar', trim: false), 
				text(name: 'JARS_A_REEMPLAZAR', defaultValue: '', description: 'Los nombres de los jars a reemplazar en las respectivas librerias compartidas', trim: false), 
                string(name: 'EMPEZAR_DESDE_LA_TAREA_NUMERO', defaultValue: '1', description: 'Indica desde que numero de tarea empezar. La tarea de leer propiedades no se puede saltar ' + 
				             '1= Compilar, 2=Desinstalar applicaciones, 3=Reemplazar Jars, 4=Reiniciar aplicaciones, 5=Desplegar, 6=Versionar, 7=Limpiar', trim: false),
            ])
        ]
    )
    def scmVars
	def jenkins_pipeline_script_prefix='@script'
	def build_workspace = "${env.WORKSPACE}${jenkins_pipeline_script_prefix}"
	def build_props_file_directory='cavaliweb_build_files'
	def jars_to_replace_base_directory='jars_to_replace/'
	def ears_to_deploy_directory='ears_to_deploy/'
	def stash_credentials_id = params.ID_DE_CREDENCIALES_DE_REPO_DE_PARCHE
	def stash_repo_path = params.RUTA_DE_REPO_DE_PARCHE
	def stash_branch = params.BRANCH_DEL_REPO_DE_PARCHE_A_USAR
	def configuration_file_path = params.RUTA_DEL_ARCHIVO_DE_CONFIGURACION_EN_EL_REPO_DE_PARCHE
	def build_props
	def config_key_value_groups_sep='|||'
	def config_key_value_groups_split='\\|\\|\\|'
	def config_key_value_group_sep='||'
	def config_key_value_group_split='\\|\\|'
	def config_key_value_sep='|'
	def config_key_value_split='\\|'
	def jdk_tool_name
	def maven_tool_name
	def amount_of_times_to_check_app_status_to_start 
	def interval_between_checks_of_app_status_to_start_in_milliseconds
	def jenkins_ip_is_the_same_as_the_deploy_server_or_servers
	def maven_compilation_flags
    def git_source_repo_path
	def git_credentials_id
    def only_deploy
	def dmgr_server_key_to_ip = [:]
	def dmgr_server_key_to_ssh_port = [:]
	def dmgr_server_key_to_credentials_id = [:]
	def dmgr_server_key_to_profile_path = [:]
	def dmgr_server_key_to_artifact_directory = [:]
    def dmgr_server_key_to_deploy_scripts_directory = [:]
	def dmgr_server_key_to_wsadmin_credentials_file_path = [:]
	
	def shared_library_key_to_ip = [:]
	def shared_library_key_to_ssh_port = [:]
	def shared_library_key_to_credentials_id = [:]
    def app_name_to_deploy_config = [:]
	def app_name_to_topology = [:]
	def app_name_to_shared_libraries = [:]
	def shared_library_to_app_names = [:]
	def jar_name_to_shared_libraries = [:]
	def shared_library_to_container_obj_name = [:]
	def shared_library_to_container_obj_type = [:]
	def shared_library_with_container_to_cached_path_on_server = [:]
	def app_names_to_ear_names = [:]
	def app_names_to_deploy
	def jar_names_to_replace
	def start_from_task_number
	def upload_artifacts_to_artifactory
	def artifactory_server_id
	def jenkins_server_and_deploy_server_are_the_same 
	def current_task_number = 0;
    stage('Leer Propiedades'){
		dir (build_workspace){
			if (stash_credentials_id == null || stash_credentials_id.isEmpty()) {
				checkout([$class: 'GitSCM', 
					branches: [[name: stash_branch]], 
					doGenerateSubmoduleConfigurations: false, 
					extensions: [
						[$class: 'CleanCheckout'],
						[$class: 'RelativeTargetDirectory', relativeTargetDir: build_props_file_directory]
					], 
					submoduleCfg: [], 
					userRemoteConfigs: [
						[url: stash_repo_path]
					]
				])
			} else {
				checkout([$class: 'GitSCM', 
					branches: [[name: stash_branch]], 
					doGenerateSubmoduleConfigurations: false, 
					extensions: [
						[$class: 'CleanCheckout'],
						[$class: 'RelativeTargetDirectory', relativeTargetDir: build_props_file_directory]
					], 
					submoduleCfg: [], 
					userRemoteConfigs: [
						[url: stash_repo_path, credentialsId: stash_credentials_id]
					]
				])
			}			
		}

    	build_props = readProperties file: build_workspace + '/' + build_props_file_directory + '/' + configuration_file_path
    	git_source_repo_path = build_props.RUTA_DE_REPOSITORIO_GIT;
        dmgr_servers_info_str = build_props.INFORMACION_DE_SERVIDORES_DMGR
		dmgr_servers_info = dmgr_servers_info_str.split(config_key_value_groups_split)
        dmgr_servers_info.each{ dmgr_server_info ->
          def dmgr_server_config = dmgr_server_info.split(config_key_value_group_split)
          // the dmgr server name is used as a key for the map
		  def dmgr_server_key
          def server_ip
		  def ssh_server_port
		  def server_credentials_id
		  for (int i = 0; i < dmgr_server_config.length; i++) {
		    key_value_config = dmgr_server_config[i].split(config_key_value_split)
			key = key_value_config[0]
			value = key_value_config[1]
			switch (key){
			  case 'nombre_servidor': dmgr_server_key = value; break;
			  case 'ip_servidor': server_ip = value; break;
			  case 'puerto_ssh_servidor': ssh_server_port = value; break;
			  case 'id_credenciales_servidor': server_credentials_id = value; break;
			  default : echo "no se considerara ${key} - ${value}"; break;
			}
		  }
		  dmgr_server_key_to_ip.put(dmgr_server_key,server_ip)
		  dmgr_server_key_to_ssh_port.put(dmgr_server_key,ssh_server_port)
		  dmgr_server_key_to_credentials_id.put(dmgr_server_key,server_credentials_id)
		}

        dmgr_servers_paths_info_str = build_props.INFORMACION_DE_RUTAS_DE_SERVIDORES_DMGR
		dmgr_servers_paths_info = dmgr_servers_directory_info_str.split(config_key_value_groups_split)
        dmgr_servers_paths_info.each{ dmgr_server_paths_info ->
          def dmgr_server_paths_config = dmgr_server_paths_info.split(config_key_value_group_split)
		  def dmgr_server_key
          def dmgr_profile_path
		  def artifact_directory
		  def deploy_scripts_directory
          def wsadmin_credentials_file_path
		  for (int i = 0; i < dmgr_server_paths_config.length; i++) {
		    key_value_config = dmgr_server_paths_config[i].split(config_key_value_split)
			key = key_value_config[0]
			value = key_value_config[1]
			switch (key){
			  case 'nombre_servidor': dmgr_server_key = value; break;
			  case 'dir_perfil_dmgr': dmgr_profile_path = value; break;
			  case 'dir_artefactos': artifact_directory = value; break;
			  case 'dir_scripts_despliegue': deploy_scripts_directory = value; break;
              case 'ruta_cred_wsadmin': wsadmin_credentials_path = value; break;
			  default : echo "no se considerara ${key} - ${value}"; break;
			}
		  }
		  dmgr_server_key_to_profile_path.put(dmgr_server_key,dmgr_profile_path)
		  dmgr_server_key_to_artifact_directory.put(dmgr_server_key,artifact_directory)
		  dmgr_server_key_to_deploy_scripts_directory.put(dmgr_server_key,deploy_scripts_directory)
		  dmgr_server_key_to_wsadmin_credentials_file_path.put(dmgr_server_key,wsadmin_credentials_file_path)
		}
        app_names_with_topologies_str = build_props.TOPOLOGIAS_PARA_DESPLIEGUE_DE_EARS
		app_names_with_topologies = app_names_with_topologies_str.split(config_key_value_groups_split)
        app_names_with_topologies.each{ app_name_with_topology ->
          app_name_with_topology_config = app_name_with_topology.split(config_key_value_group_split)
		  def app_name
		  def ear_name
          def dmgr_server_key
		  def installation_type
		  def topology = ''
		  for (int i = 0; i < app_name_with_topology_config.length; i++) {
		    key_value_config = app_name_with_topology_config[i].split(config_key_value_split)
			key = key_value_config[0]
			value = key_value_config[1]
			switch (key){
			  case 'nombre_app': app_name = value; break;
			  case 'nombre_ear': ear_name = value; break;
              case 'nombre_servidor_dmgr': dmgr_server_key = value; break;
			  case 'servidor_app':
			  case 'nodo_app':
			  case 'cluster_app':
			  case 'celda_app':
              case 'servidor_ihs':
              case 'nodo_ihs':
              case 'celda_ihs':
			    topology += app_name_with_topology_config[i] + config_key_value_group_sep; break;
              case 'tipo_instalacion': installation_type = value; break;
			  default : echo "no se considerara ${key} - ${value}"; break;
			}
		  }
          def deploy_config = ['topology':topology, 'dmgr_server_key':dmgr_server_key, 'ear_name':ear_name, 'installation_type': installation_type]
		  def deploy_configs = app_name_to_deploy_config.get(app_name)
		  if (deploy_configs == null){
		    deploy_configs = []
		    deploy_configs.add(deploy_config)
		    app_name_to_deploy_config.put(shared_library_name,deploy_configs)
		  } else {
		    deploy_configs.add(deploy_config)
		  }
		}
		
		app_names_with_shared_libraries_str = build_props.NOMBRE_DE_APLICACIONES_CON_LIBRERIAS_COMPARTIDAS
        app_names_with_shared_libraries = app_names_with_shared_libraries_str.split(config_key_value_groups_split)
        app_names_with_shared_libraries.each{ app_name_with_shared_libraries ->
		  app_name_with_shared_library_config = app_name_with_shared_libraries.split(config_key_value_group_split)
		  def shared_libraries = []
		  def shared_library_name = ''
		  for (int i = 0; i < app_name_with_shared_library_config.length; i++) {
		    key_value_config = app_name_with_shared_library_config[i].split(config_key_value_split)
			key = key_value_config[0]
			value = key_value_config[1]
			switch (key){
			  case 'nombre_app': app_name = value; break;
			  case 'nombre_libreria': shared_library_name = value; break;
			  default : echo "no se considerara ${key} - ${value}"; break;
			}
		  }
		  def app_names = shared_library_to_app_names.get(shared_library_name)
		  if (app_names == null){
		    app_names = [].toSet()
		    app_names.add(app_name)
		    shared_library_to_app_names.put(shared_library_name,app_names)
		  }else{
		    app_names.add(app_name)
		  }
		  shared_libraries.add(shared_library_name)
		  app_name_to_shared_libraries.put(app_name, shared_libraries)
		}

        //ihs_server_to_reference = build_props.SERVIDOR_IHS_A_REFERENCIAR
        //ihs_node_to_reference = build_props.NODO_IHS_A_REFERENCIAR
        //ihs_cell_to_reference = build_props.CELDA_IHS_A_REFERENCIAR
        
        //dmgr_server_ip = build_props.IP_DE_SERVIDOR_DMGR
		//dmgr_ssh_server_port = build_props.PUERTO_SSH_DE_SERVIDOR_DMGR.toInteger()
        //dmgr_server_credentials_id = build_props.ID_DE_SERVIDOR_CREDENCIALES_DE_DMGR
		
		shared_libraries_info_str = build_props.INFORMACION_DE_LIBRERIAS_COMPARTIDAS
		shared_libraries_info = shared_libraries_info_str.split(config_key_value_groups_split)
		shared_libraries_info.each{ shared_library_info ->
          shared_library_info_config = shared_library_info.split(config_key_value_group_split)
		  def shared_library_name = ''
		  def container_obj_name = ''
		  def container_obj_type = ''
		  def server_ip = ''
		  def server_credentials_id = ''
		  def server_ssh_port = ''
		  for (int i = 0; i < shared_library_info_config.length; i++) {
		    key_value_config = shared_library_info_config[i].split(config_key_value_split)
			key = key_value_config[0]
			value = key_value_config[1]
			switch (key){
			  case 'nombre_libreria': shared_library_name = value; break;
			  case 'servidor_int': server_name = value; break;
			  case 'puerto_ssh_servidor': server_ssh_port = value.toInteger(); break;
			  case 'nombre_obj_contenedor': container_obj_name = value; break;
			  case 'tipo_obj_contenedor': container_obj_type = value; break;
			  case 'ip_servidor': server_ip = value; break;
			  case 'id_credenciales_servidor': server_credentials_id = value; break;
			  default : echo "no se considerara ${key} - ${value}"; break;
			}
		  }
		  def shared_library_key = "${container_obj_name}_${container_obj_type}_${shared_library_name}"
		  shared_library_to_container_obj_name.put(shared_library_name, container_obj_name)
		  shared_library_to_container_obj_type.put(shared_library_name, container_obj_type)
		  shared_library_key_to_ip.put(shared_library_key, server_ip)
		  shared_library_key_to_ssh_port.put(shared_library_key, server_ssh_port)
		  shared_library_key_to_credentials_id.put(shared_library_key, server_credentials_id)
		}
		
		
		jars_to_shared_libraries_str = build_props.JARS_COMUNES_A_SUS_LIBRERIAS_COMPARTIDAS
		jars_to_shared_libraries = jars_to_shared_libraries_str.split(config_key_value_groups_split)
		jars_to_shared_libraries.each{ jar_to_shared_libraries ->
          shared_library_info_config = jar_to_shared_libraries.split(config_key_value_group_split)
		  def shared_libraries = []
		  def jar_name = ''
		  for (int i = 0; i < shared_library_info_config.length; i++) {
		    key_value_config = shared_library_info_config[i].split(config_key_value_split)
			key = key_value_config[0]
			value = key_value_config[1]
			switch (key){
			  case 'nombre_libreria': shared_libraries.add(value); break;
			  case 'nombre_jar': jar_name = value; break;
			  default : echo "no se considerara ${key} - ${value}"; break;
			}
		  }
		  jar_name_to_shared_libraries.put(jar_name, shared_libraries)
		}
		
        maven_compilation_flags = build_props.MAVEN_COMPILATION_FLAGS
		
		
		app_names_tp_deploy_array = params.NOMBRES_DE_APLICACIONES_A_DESPLEGAR.split('\n')
		app_names_to_deploy = []
		app_names_tp_deploy_array.each {app_names_to_deploy.add(it)}
		
		jars_to_replace_str = params.JARS_A_REEMPLAZAR
		jar_names_to_replace = jars_to_replace_str.split('\n')
		
		jdk_tool_name=build_props.NOMBRE_DE_JDK_CONFIGURADO
	    maven_tool_name=build_props.NOMBRE_DE_MAVEN_CONFIGURADO
		amount_of_times_to_check_app_status_to_start = build_props.CANTIDAD_DE_REVISIONES_DEL_ESTADO_DE_LA_APLICACION
        interval_between_checks_of_app_status_to_start_in_milliseconds = build_props.INTERVALO_DE_REVISION_DEL_ESTADO_DE_LA_APLICACION_EN_MILLISEGUNDOS
		jenkins_ip_is_the_same_as_the_deploy_server_or_servers = ("true".equals(build_props.IP_DE_JENKINS_ES_IGUAL_AL_IP_DEL_SERVIDOR_O_SERVIDORES_DE_DESPLIEGUE))
        start_from_task_number = params.EMPEZAR_DESDE_LA_TAREA_NUMERO.toInteger()
		upload_artifacts_to_artifactory = ("true".equals(build_props.SUBIR_ARTEFACTOS_AL_ARTIFACTORY))
		artifactory_server_id = build_props.ID_DE_SERVIDOR_ARTIFACTORY
		jenkins_server_and_deploy_server_are_the_same = ("true".equals(build_props.SERVIDOR_DE_JENKINS_Y_SERVIDOR_DMGR_SON_EL_MISMO))
        echo "jdk_tool_name ${jdk_tool_name}"
	    echo "maven_tool_name ${maven_tool_name}"
	    echo "maven_compilation_flags ${maven_compilation_flags}"
        echo "git_source_repo_path ${git_source_repo_path}"
	    echo "git_credentials_id ${git_credentials_id}"
	    echo "dmgr_server_key_to_ip ${dmgr_server_key_to_ip}"
	    echo "dmgr_server_key_to_ssh_port ${dmgr_server_key_to_ssh_port}"
	    echo "dmgr_server_key_to_credentials_id ${dmgr_server_key_to_credentials_id}"
	    echo "dmgr_server_key_to_profile_path ${dmgr_server_key_to_profile_path}"
		echo "dmgr_server_key_to_artifact_directory ${dmgr_server_key_to_artifact_directory}"
	    echo "dmgr_server_key_to_deploy_scripts_directory ${dmgr_server_key_to_deploy_scripts_directory}"
	    echo "dmgr_server_key_to_wsadmin_credentials_file_path ${dmgr_server_key_to_wsadmin_credentials_file_path}"
	    echo "shared_library_key_to_ip ${shared_library_key_to_ip}"
		echo "shared_library_key_to_ssh_port ${shared_library_key_to_ssh_port}"
	    echo "shared_library_key_to_credentials_id ${shared_library_key_to_credentials_id}"
		echo "app_name_to_topology ${app_name_to_topology}"
		echo "app_name_to_shared_libraries ${app_name_to_shared_libraries}"
		echo "shared_library_to_app_names ${shared_library_to_app_names}"
	    echo "jar_name_to_shared_libraries ${jar_name_to_shared_libraries}"
	    echo "shared_library_to_container_obj_name ${shared_library_to_container_obj_name}"
	    echo "shared_library_to_container_obj_type ${shared_library_to_container_obj_type}"
	    echo "app_names_to_ear_names ${app_names_to_ear_names}"
		echo "app_names_to_deploy ${app_names_to_deploy}"
	    echo "jar_names_to_replace ${jar_names_to_replace}"
		echo "def amount_of_times_to_check_app_status_to_start ${amount_of_times_to_check_app_status_to_start }"
	    echo "interval_between_checks_of_app_status_to_start_in_milliseconds ${interval_between_checks_of_app_status_to_start_in_milliseconds}"
		echo "jenkins_ip_is_the_same_as_the_deploy_server_or_servers ${jenkins_ip_is_the_same_as_the_deploy_server_or_servers}"
		echo "start_from_task_number ${start_from_task_number}"
		echo "upload_artifacts_to_artifactory ${upload_artifacts_to_artifactory}"
		echo "artifactory_server_id ${artifactory_server_id}"
		echo "jenkins_server_and_deploy_server_are_the_same ${jenkins_server_and_deploy_server_are_the_same}"
    }
    stage('Compilar'){
	  current_task_number++
      if(start_from_task_number > current_task_number){
          echo 'saltando tarea'
          return
      }
	  jdk = tool name: jdk_tool_name
	  maven = tool name: maven_tool_name
      env.JAVA_HOME = "${jdk}"
	  env.M2_HOME = "${maven}"
	  sh "'$M2_HOME'/bin/mvn ${maven_compilation_flags} clean package -f '${build_workspace}'/wari-migration/pom.xml"
	}
    stage('Desinstalar Aplicaciones'){
      current_task_number++
      if(start_from_task_number > current_task_number){
        echo 'saltando tarea'
        return
      }
	  def apps_to_uninstall = app_name_to_topology.findAll{entry -> app_names_to_deploy.contains(entry.key)}
	  def dmgr_server_keys_to_apps_to_uninstall = [:]
      apps_to_uninstall.each { app_to_install_name, deploy_configurations ->
        def dmgr_server_key = deploy_configurations.get('dmgr_server_key')
        def app_config = ['app_name':app_to_install_name, 'topology':deploy_configurations.get('topology')]
        def app_uninstall_config = dmgr_server_keys_to_apps_to_uninstall.get(dmgr_server_key)
        if (app_uninstall_config == null){
		  app_uninstall_config = []
          app_uninstall_config.add(app_config)
          dmgr_server_keys_to_apps_to_uninstall.put(dmgr_server_key,app_uninstall_config)
        } else {
          app_uninstall_config.add(app_config)
        }
	  }
      def are_there_apps_to_uninstall = !apps_to_uninstall.isEmpty() && !dmgr_server_keys_to_apps_to_uninstall.isEmpty()
	  if(!are_there_apps_to_uninstall){
        echo "no hay ninguna aplicacion configurada para desinstalar"
		return
	  }
	  dmgr_server_keys_to_apps_to_uninstall.each{ dmgr_server_key, apps_to_uninstall_configs ->
	    apps_with_topologies_to_uninstall = apps_to_uninstall_configs.inject(''){
	      str, app_to_uninstall -> str + 'nombre_app' + config_key_value_sep + app_to_uninstall.get('app_name') + config_key_value_group_sep + app_to_uninstall.get('topology') + config_key_value_groups_sep
	    }
	    // using string.length is discouraged in pipelines, but String.lastIndexOf("") works like length
	    apps_with_topologies_to_uninstall = apps_with_topologies_to_uninstall.substring(0,apps_with_topologies_to_uninstall.lastIndexOf("") - 3)
	    echo "apps_with_topologies_to_uninstall $apps_with_topologies_to_uninstall"
        def dmgr_server_credentials_id = dmgr_server_key_to_credentials_id.get(dmgr_server_key)
        def dmgr_server_ip = dmgr_server_key_to_ip.get(dmgr_server_key)
        def dmgr_ssh_server_port = dmgr_server_key_to_ssh_port.get(dmgr_server_key)
        def dmgr_deploy_scripts_directory = dmgr_server_key_to_deploy_scripts_directory.get(dmgr_server_key)
        def wsadmin_credential_properties_file_path = dmgr_server_key_to_wsadmin_credentials_file_path.get(dmgr_server_key)
        def dmgr_profile_path = dmgr_server_key_to_profile_path.get(dmgr_server_key)
		if(jenkins_server_and_deploy_server_are_the_same){
          sh "'${dmgr_deploy_scripts_directory}desinstalacionDeEARS.sh' "+
             "-dmgrProfilePath '${dmgr_profile_path}' "+
             "-wsadminPropertiesFile '${wsadmin_credential_properties_file_path}' "+
             "-appNamesAndTopologies '${apps_with_topologies_to_uninstall}' "
		}else{
		  withCredentials([usernamePassword(credentialsId: dmgr_server_credentials_id, passwordVariable: 'password', usernameVariable: 'user')]) {
	        def remote = [:]
            remote.name = dmgr_server_key
            remote.host = dmgr_server_ip
		    remote.port = dmgr_ssh_server_port
            remote.user = user
            remote.password = password
            remote.allowAnyHosts = true
            sshCommand remote: remote, command: "'${dmgr_deploy_scripts_directory}desinstalacionDeEARS.sh' "+
              "-dmgrProfilePath '${dmgr_profile_path}' "+
              "-wsadminPropertiesFile '${wsadmin_credential_properties_file_path}' "+
              "-appNamesAndTopologies '${apps_with_topologies_to_uninstall}' "
	      }	
		}
      }
    }
    stage('Desplegar Aplicaciones'){
	  current_task_number++
	  if(start_from_task_number > current_task_number){
        echo 'saltando tarea'
        return
      }
	  def apps_to_install = app_name_to_deploy_config.findAll{entry -> app_names_to_deploy.contains(entry.key)}
	  def dmgr_server_keys_to_apps_to_install = [:]
      apps_to_install.each { app_to_install_name, deploy_configurations ->
        def dmgr_server_key = deploy_configurations.get('dmgr_server_key')
        def app_config = {'app_name':app_to_install_name, 'topology':deploy_configurations.get('topology'), 'ear_name':deploy_configurations.get('ear_name'), 'installation_type': deploy_configurations.get('installation_type') }
        def app_install_config = dmgr_server_keys_to_apps_to_install.get(dmgr_server_key)
        if (app_install_config == null){
		  app_install_config = []
          app_install_config.add(app_config)
          dmgr_server_keys_to_apps_to_install.put(dmgr_server_key,app_install_config)
        } else {
          app_install_config.add(app_config)
        }
	  }
	  echo "apps_to_install: ${apps_to_install}"
      echo "dmgr_server_keys_to_apps_to_install: ${dmgr_server_keys_to_apps_to_install}"
      def are_there_apps_to_install = !apps_to_install.isEmpty() && !dmgr_server_keys_to_apps_to_install.isEmpty()
	  if(!are_there_apps_to_install){
        echo "no hay ninguna aplicacion configurada para desplegar"
		return
	  }
	  def ear_paths_to_move = [].toSet()
	  dir (build_workspace) {
	    apps_to_install.each { app_name, app_configs ->
          app_configs.each { app_config ->
	        def ear_name = app_config.get('ear_name')
	        def ear_files = findFiles glob: "**/${ear_name}"
			ear_paths_to_move.add("'"+ear_files[0].path+"'")
	      }
	    }
      }
	  def paths_to_copy = ear_paths_to_move.join(' ')
	  sh "cd '${build_workspace}/'; mkdir -p '${ears_to_deploy_directory}'; mv -f ${paths_to_copy} '${ears_to_deploy_directory}'"
	  dmgr_server_keys_to_apps_to_install.each{ dmgr_server_key, apps_to_install_configs ->
	    def install_configurations_of_apps = ''
	    def kv_sep=config_key_value_sep
	    def kv_grp_sep=config_key_value_group_sep
	    apps_to_install_configs.each { app_name, app_to_install_config ->
	      def config_to_add = "nombre_app${kv_sep}${app_name}${kv_grp_sep}"
		  if(jenkins_server_and_deploy_server_are_the_same){
			config_to_add += "nombre_ear${kv_sep}${dmgr_artifact_directory}" + app_to_install_config.get('ear_name') + "${kv_grp_sep}"  
		  }else{
			config_to_add += "nombre_ear${kv_sep}${build_workspace}/${ears_to_deploy_directory}" + app_to_install_config.get('ear_name') + "${kv_grp_sep}"
		  }
          config_to_add += "tipo_instalacion${kv_sep}" + app_to_install_config.get('installation_type') + "${kv_grp_sep}"
		  config_to_add += app_to_install_config.get('topology')
		  config_to_add += "${kv_grp_sep}libreria_compartida${kv_sep}" + app_name_to_shared_libraries.get(app_name).join(config_key_value_group_sep + 'libreria_compartida' + config_key_value_sep)
		  config_to_add += config_key_value_groups_sep
		  install_configurations_of_apps += config_to_add
	    }
	    echo "install_configurations_of_apps: ${install_configurations_of_apps}"
	    install_configurations_of_apps = install_configurations_of_apps.substring(0,install_configurations_of_apps.lastIndexOf("") - 3)
        def dmgr_server_credentials_id = dmgr_server_key_to_credentials_id.get(dmgr_server_key)
        def dmgr_server_ip = dmgr_server_key_to_ip.get(dmgr_server_key)
        def dmgr_ssh_server_port = dmgr_server_key_to_ssh_port.get(dmgr_server_key)
        def dmgr_artifact_directory = dmgr_server_key_to_artifact_directory.get(dmgr_server_key)
        def dmgr_deploy_scripts_directory = dmgr_server_key_to_deploy_scripts_directory.get(dmgr_server_key)
        def wsadmin_credential_properties_file_path = dmgr_server_key_to_wsadmin_credentials_file_path.get(dmgr_server_key)
        def dmgr_profile_path = dmgr_server_key_to_profile_path.get(dmgr_server_key)
		if(jenkins_server_and_deploy_server_are_the_same){
          sh "'${dmgr_deploy_scripts_directory}despliegueDeEARSConReferenciaIHS.sh' " +
	         "-dmgrProfilePath '${dmgr_profile_path}' "+
	         "-wsadminPropertiesFile '${wsadmin_credential_properties_file_path}' "+
	         "-amountOfTimesToQueryAppState ${amount_of_times_to_check_app_status_to_start} " +
	         "-appStateQueryIntervalInMilliseconds ${interval_between_checks_of_app_status_to_start_in_milliseconds} " +
	         "-installConfigurationsOfApps '${install_configurations_of_apps}' "
		}else{
	      withCredentials([usernamePassword(credentialsId: dmgr_server_credentials_id, passwordVariable: 'password', usernameVariable: 'user')]) {
	        def remote = [:]
            remote.name = dmgr_server_key
            remote.host = dmgr_server_ip
	        remote.port = dmgr_ssh_server_port
            remote.user = user
            remote.password = password
            remote.allowAnyHosts = true
	        sshPut remote: remote, from: "${build_workspace}/${ears_to_deploy_directory}", into: "${dmgr_artifact_directory}"
	        sshCommand remote: remote, failOnError :true ,command: "mv -f '${dmgr_artifact_directory}/${ears_to_deploy_directory}'*.ear '${dmgr_artifact_directory}'; rm -rf '${dmgr_artifact_directory}/${ears_to_deploy_directory}'"
	      }
	      withCredentials([usernamePassword(credentialsId: dmgr_server_credentials_id, passwordVariable: 'password', usernameVariable: 'user')]) {
	        def remote = [:]
            remote.name = dmgr_server_key
            remote.host = dmgr_server_ip
	        remote.port = dmgr_ssh_server_port
            remote.user = user
            remote.password = password
            remote.allowAnyHosts = true
            sshCommand remote: remote, failOnError :true ,command: "'${dmgr_deploy_scripts_directory}despliegueDeEARSConReferenciaIHS.sh' " +
	        "-dmgrProfilePath '${dmgr_profile_path}' "+
	        "-wsadminPropertiesFile '${wsadmin_credential_properties_file_path}' "+
	        "-amountOfTimesToQueryAppState ${amount_of_times_to_check_app_status_to_start} " +
	        "-appStateQueryIntervalInMilliseconds ${interval_between_checks_of_app_status_to_start_in_milliseconds} " +
	        "-installConfigurationsOfApps '${install_configurations_of_apps}' "
	      }
		}
      }
    }
	stage('Versionar con Artifactory'){
	  current_task_number++
	  if(start_from_task_number > current_task_number){
        echo 'saltando tarea'
        return
      }
	  if (upload_artifacts_to_artifactory) {
	    dir (build_workspace){
		  def now_timestamp = new java.text.SimpleDateFormat('YYYY_MM_dd_HH_mm_ss').format(new java.util.Date())
		  def server = Artifactory.server artifactory_server_id
	      // Create the upload spec, which is in json format
          def uploadSpec = """
		  {
              "files": [
                      {
                          "pattern": "${ears_to_deploy_directory}*.ear",
                          "target": "CAVALI_DEPLOYS/CAVALIWEB/DESARROLLO/EARS/${now_timestamp}/"
                      }"""
          shared_library_key_to_ip.each{ entry ->
            uploadSpec += """
		    ,{
		  	  "pattern": "${jars_to_replace_base_directory}${entry.key}/*.jar",
		  	  "target": "CAVALI_DEPLOYS/CAVALIWEB/DESARROLLO/JARS/${now_timestamp}/"
            }"""
		  }
		  uploadSpec += """]
          }"""
		  echo "uploadSpec: ${uploadSpec}"
		  echo "server ${server}"
          // Upload to Artifactory using the spec
          uploadInfo = server.upload spec: uploadSpec
		  echo "info de artifactory ${uploadInfo}"
		}		
	  } else {
		echo "el parametro SUBIR_ARTEFACTOS_AL_ARTIFACTORY esta desactivado, no se subira artefactos al artifactory"
	  }
	}
    stage('Limpiar repositorio'){
		current_task_number++
		if(start_from_task_number > current_task_number){
          echo 'saltando tarea'
          return
        }
        cleanWs deleteDirs: true, notFailBuild: true
    }
}
