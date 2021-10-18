/**
* Resumen.
* Objeto: Jenkinsfile
* Descripcion: El objeto Jenkinsfile contiene el script de pipeline para
* el despliegue de Factrack mediante Jenkins
* Fecha de Creacion: 16/01/2021
* Requerimiento de Creacion: REV-1173
* Autor: HRL
*/

//Notes: 
// using string.length is discouraged in pipelines, but String.lastIndexOf("") works like length, but if at all posible, dont use it

// all the values that come from the properties file are parsed as maps where a key of the map points to a collection (usually a list) of maps with the key pairs
// in the properties file, e.g:
// { 'a' : [{prop_1:'a',prop_2:'value'}], 'b' : [{prop_1:'b',prop_2:'value2'}]}
// this will enable to map multilple configurations to one key and if needed, map several values to one key

// the script has 3 sections
// the utility methods, for generic use
// the pipeline step methods, for use in the stages
// the node and steps section, where the pipeline gets exectuted
// this is done due to de 64KB limit on methods
// only the read properties section is directly on the node and steps section due the global variables
/**
Utility Methods
**/

def addItemToCollectionInMapValue (item_to_add, key_to_add_to, map_to_add_item_to, new_collection_instance) {
  def existing_collection_for_key = map_to_add_item_to.get(key_to_add_to)
  if(existing_collection_for_key == null){
    if (item_to_add instanceof java.util.Collection){
      map_to_add_item_to.put(key_to_add_to, item_to_add)
    }else{
      existing_collection_for_key = new_collection_instance.clone()
      existing_collection_for_key.add(item_to_add)
      map_to_add_item_to.put(key_to_add_to, existing_collection_for_key)
    }
  }else{
    if (item_to_add instanceof java.util.Collection){
      existing_collection_for_key.addAll(item_to_add)
    }else{
      existing_collection_for_key.add(item_to_add)
    }
  }
  return map_to_add_item_to;
}

def mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects (key_and_value_pair_groups_str, prop_file_key_name_to_map_key_name, 
    prop_file_key_to_use_as_result_map_key, key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex) {
  def key_field_to_mappings_result = [:]
  //echo "$key_and_value_pair_groups_str"
  def key_and_value_pair_groups = key_and_value_pair_groups_str.split(key_value_pair_groups_split_regex)
  for (int i = 0; i < key_and_value_pair_groups.length; i++) {
    //echo "$key_and_value_pair_groups[i]"
    if(key_and_value_pair_groups[i] == null){
       echo "error, se encontro un null al partir con $key_value_pair_groups_split_regex la cadena $key_and_value_pair_groups_str"
    }
    def key_and_value_pairs = key_and_value_pair_groups[i].split(key_value_pair_split_regex)
    def result_mapping = [:]
    def mapping_key
    for (int j = 0; j < key_and_value_pairs.length; j++) {
      //echo "${key_and_value_pairs[j]}"
      if(key_and_value_pairs[j] == null){
        echo "error, se encontro un null al partir con $key_value_pair_split_regex la cadena ${key_and_value_pair_groups[i]}"
      }
      def key_value_config = key_and_value_pairs[j].split(key_value_split_regex)
      def key = (key_value_config.length > 0) ? key_value_config[0] : ''
      def value = (key_value_config.length > 1) ? key_value_config[1] : ''
      if(prop_file_key_to_use_as_result_map_key.equals(key)){
        mapping_key = value;
      }
      if(prop_file_key_name_to_map_key_name.get(key) != null){
        result_mapping.put(prop_file_key_name_to_map_key_name.get(key), value);
      }else{
        echo "no se considerara ${key} - ${value}";
      }
    }
    addItemToCollectionInMapValue(result_mapping, mapping_key, key_field_to_mappings_result, [])
  }
  return key_field_to_mappings_result;
}

def joinObjectsIntoConfigurationString (list_of_objects_to_join, map_of_obj_idx_to_property_closures, key_value_sep, key_value_pair_sep) {
  def key_value_pairs = []
  list_of_objects_to_join.eachWithIndex{ object_to_join, idx ->
    def property_closures = map_of_obj_idx_to_property_closures.get(idx)
    property_closures.each{ property_closure ->
      def value_to_add = property_closure(object_to_join, key_value_sep, key_value_pair_sep);
      if (value_to_add != null && !value_to_add.isEmpty()) {
        key_value_pairs.add(value_to_add)
      }
    }
  }
  def joined_key_value_pairs = ''
  for (int i = 0; i < key_value_pairs.size(); i++){
    joined_key_value_pairs += key_value_pairs[i]
    if( i < key_value_pairs.size() - 1){
      joined_key_value_pairs += key_value_pair_sep
    }
  }
  return joined_key_value_pairs;
}

def joinMapKeysAndValuesIntoString (map_to_join, key_value_sep, key_value_pair_sep) {
  def keys_and_values_iterator = map_to_join.entrySet().iterator();
  def str = ''
  while(keys_and_values_iterator.hasNext()){
    def key_and_value = keys_and_values_iterator.next();
    def kv_pair_sep = ( keys_and_values_iterator.hasNext() )? key_value_pair_sep : ''
    str += "${key_and_value.key}${key_value_sep}${key_and_value.value}${kv_pair_sep}"
  }
  return str;
}

def executeShellCommandOnServer(cmd_to_execute, server_credentials_id, server_ip, ssh_server_port, jenkins_server_and_target_server_are_the_same, local_error_msg){
  if(jenkins_server_and_target_server_are_the_same){
    def sh_status = sh returnStatus: true, script: cmd_to_execute
    if (sh_status != 0){
      error(local_error_msg)
    }
  }else{
    withCredentials([usernamePassword(credentialsId: server_credentials_id, passwordVariable: 'password', usernameVariable: 'user')]) {
      def remote = ['name':server_ip, 'host':server_ip, 'port':ssh_server_port.toInteger(), 'user':user, 'password':password, 'allowAnyHosts':true]
      sshCommand remote: remote, failOnError: true, command: cmd_to_execute
    }   
  }
}

def prepProyectForSonarQubeScan(maven_pom_file_path, build_workspace, jdk_tool_name, maven_tool_name, sq_project_info, git_repo_info, sonar_pom_properties_override){
def now = new Date().format('yyyy-MM-dd_HH.mm.ss')
echo """====================================================================================================
INICIO DE PREPARACION DE CODIGO PARA ESCANEO ${now}
===================================================================================================="""
  def maven_pom_file_full_path = (maven_pom_file_path != null && !maven_pom_file_path.isEmpty()) ? "${build_workspace}/${maven_pom_file_path}" : ""
  echo "maven_pom_file_full_path ${maven_pom_file_full_path}"
    jdk = tool name: jdk_tool_name
    maven = tool name: maven_tool_name
    env.JAVA_HOME = "${jdk}"
    env.M2_HOME = "${maven}"
  def version_for_sonar_scan = git_repo_info.lastestTag//(current_version.equals("")) ? 'no-version' : current_version
  if(!git_repo_info.branch){
    echo "No se pudo determinar la rama del proyecto, se usara el commit como branch"
  }
  def repo_branch = (git_repo_info.branch) ? git_repo_info.branch : git_repo_info.commitId
  def production_branch = repo_branch.equals(git_repo_info.productionBranch)
  if(production_branch){
    current_branch = ''
  }else{
    def sep_idx = repo_branch.lastIndexOf('/');
    if(sep_idx == -1){
        current_branch = repo_branch
    }else{
        current_branch = repo_branch.substring(sep_idx+1)
    }
  }
  
  def properties_to_write = ['sonar.projectVersion':version_for_sonar_scan, 'sonar.projectKey':sq_project_info.projectKeyWithoutBranch ,
     'sonar.branch':sq_project_info.projectBranch, 'sonar.moduleKey': '\\${project.groupId}:\\${project.artifactId}'];
  if (sonar_pom_properties_override){
	properties_to_write.putAll(sonar_pom_properties_override);
  }
  sh '''PARENT_POM_FOR_SCAN="'''+maven_pom_file_full_path+'''"
      PROPERTIES_TAG_PRESENT=$(grep -m 1 -E \'<properties>\' "$PARENT_POM_FOR_SCAN" || true)
      if [ -z "$PROPERTIES_TAG_PRESENT"  ]; then
        sed -i -E "s#</project>#\\t<properties>\\n\\t</properties>\\n</project>\\n#g" "$PARENT_POM_FOR_SCAN"
      fi
      '''
  properties_to_write.each{ property_to_write ->
    def pom_property = sh returnStdout: true, script: "set +e; grep '<${property_to_write.key}>' '${maven_pom_file_full_path}'; exit 0;"
    if(pom_property){
      echo "la propiedad ${property_to_write.key} ya existe, con la siguiente configuracion ${pom_property}"
    }else{
      echo "la propiedad ${property_to_write.key} no existe, se añadira con el valor ${property_to_write.value}"
      sh "sed -i -E \"s#.*?<properties>.*?#\\t<properties>\\n\\t\\t<${property_to_write.key}>${property_to_write.value}</${property_to_write.key}>#g\" '${maven_pom_file_full_path}'"
    }
  }
  now = new Date().format('yyyy-MM-dd_HH.mm.ss')
echo """====================================================================================================
FIN DE PREPARACION DE CODIGO PARA ESCANEO ${now}
===================================================================================================="""
  return true
}

def getProjectInfoFromPom(build_workspace, jdk_tool_name, maven_tool_name, maven_pom_file_path) {
  def result = [:]
  dir (build_workspace) {
	def jdk = tool name: jdk_tool_name
    def maven = tool name: maven_tool_name
    env.JAVA_HOME = "${jdk}"
    env.M2_HOME = "${maven}"
    def group_id = sh script: "'$M2_HOME'/bin/mvn org.apache.maven.plugins:maven-help-plugin:3.2.0:evaluate -Dexpression=project.groupId -q -DforceStdout -f '${maven_pom_file_path}'", returnStdout: true
    def artifact_id = sh script: "'$M2_HOME'/bin/mvn org.apache.maven.plugins:maven-help-plugin:3.2.0:evaluate -Dexpression=project.artifactId -q -DforceStdout -f '${maven_pom_file_path}'", returnStdout: true
	result.groupId = group_id.replaceAll('\n','')
	result.artifactId = artifact_id.replaceAll('\n','')
  }
  return result;
}

def getGitRepositoryInfo(build_workspace, production_branch) {
  def result = [:]
  dir (build_workspace) {
    def commit_id = sh returnStdout: true, script: 'git --no-pager rev-parse HEAD'
	commit_id.replaceAll('\n','')
    def repo_branch_cmd_output = sh returnStdout: true, script: "git --no-pager show-ref | grep ${commit_id}";
	def repo_branch_info = repo_branch_cmd_output.split(' ')
	if (repo_branch_info.length > 1){
	  result.branch = repo_branch_info[1].replaceAll('\n','');	
	}
	result.commitId = commit_id;
	def production_branch_cmd_output = sh returnStdout: true, script: "git --no-pager show-ref | grep -E '/${production_branch}\$'";
	def production_branch_info = production_branch_cmd_output.split(' ')
	if (production_branch_info.length > 1){
	  result.productionBranch = production_branch_info[0].replaceAll('\n','');
	  result.productionBranchCommitId = production_branch_info[1].replaceAll('\n','');
	}
	def latest_tag = sh returnStdout: true, script: 'git --no-pager tag --merged HEAD --sort=creatordate | tail -n 1'
	result.lastestTag = (latest_tag) ? latest_tag.replaceAll('\n','') : 'no-version'
  }
  return result;
}

def getSonarQubeProjectInfo (sonarqube_server_name, jenkins_sonarqube_credential_id, build_workspace, git_repo_info, maven_pom_info, sq_project_key_prefix, production_branch){
  dir (build_workspace) {
    withSonarQubeEnv(sonarqube_server_name){
	  def sq_url = env.SONAR_HOST_URL
	  def sonarqube_token = jenkins_sonarqube_credential_id
	  def result = [:]
      def sq_branch_for_projecy_key
      def repo_branch = git_repo_info.branch
      if (!repo_branch){
        echo "No se pudo determinar la rama del proyecto, se usara el commit como branch"
	    repo_branch = git_repo_info.commitId
      } 
      if(repo_branch.endsWith("/${production_branch}")){
        sq_branch_for_projecy_key = ''
      }else{
        def sep_idx = repo_branch.lastIndexOf('/');
	    def sq_branch = (sep_idx == -1) ? repo_branch : repo_branch.substring(sep_idx+1)
		result.projectBranch = sq_branch
        sq_branch_for_projecy_key = ':' + sq_branch
      }
      def sq_project_key_for_create = (sq_project_key_prefix) ? "${sq_project_key_prefix}:${maven_pom_info.groupId}:${maven_pom_info.artifactId}" : "${maven_pom_info.groupId}:${maven_pom_info.artifactId}"
	  echo "sq_project_key_for_create '${sq_project_key_for_create}'"
      def sq_project_key = "${sq_project_key_for_create}${sq_branch_for_projecy_key}"
	  def project_search_url = "${sq_url}/api/projects/search?projects=${sq_project_key}"
      def sq_projects_json_str = null
      def sq_projects_request = httpRequest httpMode: 'GET', authentication: "${sonarqube_token}", acceptType: 'APPLICATION_JSON_UTF8', url: project_search_url
      def sq_found_project = readJSON text: sq_projects_request.content
	  echo "sq_found_project ${sq_found_project}"
	  result.projectKey = sq_project_key
	  result.projectKeyWithoutBranch = sq_project_key_for_create
      if(sq_found_project == null || sq_found_project.components.size() == 0 ){
        result.projectExistsInSonarqube = false
      } else {
		result.projectExistsInSonarqube = true
		result.id = sq_found_project.components[0].id
	  }
	  return result
	}
  }
}

/**
Pipeline Methods
**/

def scanPreparationStep (sonarqube_server_name, jenkins_sonarqube_credential_id, build_workspace, sq_project_info, sq_project_key_prefix, sq_quality_profile_name_to_languages_for_project_creation, sp_quality_gate_name_for_project_creation, production_branch){
  dir (build_workspace) {
    withSonarQubeEnv(sonarqube_server_name){
	  def now = new Date().format('yyyy-MM-dd_HH.mm.ss')
      echo """====================================================================================================
INICIO DE CREACION DE PROYECTO SONARQUBE ${now}
===================================================================================================="""
	  def sq_url = env.SONAR_HOST_URL
	  def sonarqube_token = jenkins_sonarqube_credential_id
	  def result = [:]
	  result.putAll(sq_project_info)
      result.projectCreated = false

	  def sq_project_key_for_create = sq_project_info.projectKeyWithoutBranch
	  def sq_project_key = sq_project_info.projectKey
	  def sq_branch_parm_for_project_creation = (sq_project_info.projectBranch) ? "&branch=${sq_project_info.projectBranch}" : ''
      def encoded_sq_project_name = java.net.URLEncoder.encode("Nombre por definir para ${sq_project_key_for_create}", "UTF-8")
      def project_creation_url = "${sq_url}/api/projects/create?project=${sq_project_key_for_create}&name=${encoded_sq_project_name}${sq_branch_parm_for_project_creation}"
      def sq_project_creation_request = httpRequest httpMode: 'POST', authentication:"${sonarqube_token}", acceptType: 'APPLICATION_JSON_UTF8', url: project_creation_url
      def sq_project_creation = readJSON text: sq_project_creation_request.content
      if(sq_project_creation == null){
        echo "Hubo un error al crear el Proyecto '${encoded_sq_project_name}' con llave '${sq_project_key}', por favor verificar"
        result.error = true
        return result
      }else if(sq_project_creation.project == null){
        echo "Proyecto '${encoded_sq_project_name}' con llave '${sq_project_key}' no se creo " + 
        "satisfactoriamente, se encontro este mensaje: '${sq_project_creation}', por favor verificar"
        result.error = true
        return result
      }
      result.projectCreated =  true
	  result.id = sq_project_creation.id
      
      def qp_results = []
      def failed_quality_profiles = []
      
      sq_quality_profile_name_to_languages_for_project_creation.each{ sq_quality_profile_name, languages ->
        def encoded_sq_qp_name = java.net.URLEncoder.encode(sq_quality_profile_name, "UTF-8")
        def sq_qp_search_url = "${sq_url}/api/qualityprofiles/search?qualityProfile=${encoded_sq_qp_name}"
        def sq_qp_search_request = httpRequest httpMode: 'GET', validResponseCodes:"100:399,404", authentication:"${sonarqube_token}", acceptType: 'APPLICATION_JSON_UTF8', url: sq_qp_search_url
	    def sq_qp_search_result = readJSON text: sq_qp_search_request.content
	    if(sq_qp_search_result == null || sq_qp_search_result.profiles == null || sq_qp_search_result.profiles.size() == 0){
	      failed_quality_profiles.add(sq_quality_profile_name)
        } else {
          qp_results.add(sq_qp_search_result)
	    }
      }
      
      if(failed_quality_profiles){
        echo "No se encontraron perfiles de calidad con el nombre ${failed_quality_profiles}, por favor verificar"
        result.error = true
        return result   
      }
      def sq_qp_found_languages = [:]
      def all_languages_found =true;
      qp_results.each{ sq_qp_search_result ->
	    sq_qp_name = sq_qp_search_result.profiles[0].name
        def sq_qp_result_languages = sq_qp_search_result.profiles.findAll{sq_qp_name.equals(it['name'])}.collect{ sq_qp -> sq_qp.language}
        def missing_languages = new java.util.ArrayList(sq_quality_profile_name_to_languages_for_project_creation.get(sq_qp_name).collect{ qp_to_language -> qp_to_language.language})
	    missing_languages.removeAll(sq_qp_result_languages)
	    if(!missing_languages.isEmpty()){
	      all_languages_found = false
	    }
		sq_qp_found_languages.put(sq_qp_name, sq_qp_result_languages)
      }
      
      if(!all_languages_found){
        echo "Se encontraron los perfiles de calidad y lenguages '${sq_qp_found_languages}' cuando se esperaba "+
	    "los siguientes lenguages '${sq_quality_profile_name_to_languages_for_project_creation}' , por favor verificar"
        result.error = true
        return result
      }
      def sq_qg_id = ''
      if(sp_quality_gate_name_for_project_creation != null && !sp_quality_gate_name_for_project_creation.isEmpty()){
        def encoded_sq_qg_name = java.net.URLEncoder.encode(sp_quality_gate_name_for_project_creation, "UTF-8")
        def sq_qg_search_url = "${sq_url}/api/qualitygates/show?name=${encoded_sq_qg_name}"
        def sq_qg_search_request = httpRequest httpMode: 'GET', validResponseCodes:"100:399,404", authentication:"${sonarqube_token}", acceptType: 'APPLICATION_JSON_UTF8', url: sq_qg_search_url

        def sq_qg_search_result = readJSON text: sq_qg_search_request.content
        if(sq_qg_search_result == null || sq_qg_search_result.id == null){
            echo "No se encontraron umbrales de calidad con el nombre ${sp_quality_gate_name_for_project_creation}, por favor verificar"
            result.error = true
            return result
        }
        sq_qg_id = "${sq_qg_search_result.id}"
      }
      
      def error_during_assignment = false
      qp_results.each{ qp_result ->
	    qp_result.profiles.each{ qp_config ->
          def qp_id = qp_config.key
          def sq_qp_assign_url = "${sq_url}/api/qualityprofiles/add_project?profileKey=${qp_id}&projectKey=${sq_project_key}"
	      def sq_qp_assign_request = httpRequest httpMode: 'POST', authentication:"${sonarqube_token}", acceptType: 'APPLICATION_JSON_UTF8', url: sq_qp_assign_url
	      def sq_qp_assign_json_str = sq_qp_assign_request.content;
	      
          if(sq_qp_assign_json_str != null){
              sq_qp_assign_json_str = sq_qp_assign_json_str.trim()
          }
          if(sq_qp_assign_json_str != null && !sq_qp_assign_json_str.isEmpty()){
              echo "El perfil de calidad con id \'${qp_id}\' no se pudo asignar al projecto,"+
              " \'${sq_project_key}\' salio el siguiente mensaje: \'${sq_qp_assign_json_str}\', por favor verificar"
              error_during_assignment = true;
          }
          if(!sq_qg_id.isEmpty()){
            def sq_qg_assign_url = "${sq_url}/api/qualitygates/select?gateId=${sq_qg_id}&projectKey=${sq_project_key}"
	        def sq_qg_assign_request = httpRequest httpMode: 'POST', authentication:"${sonarqube_token}", acceptType: 'APPLICATION_JSON_UTF8', url: sq_qg_assign_url
	    	def sq_qg_assign_json_str = sq_qg_assign_request.content
            if(sq_qg_assign_json_str != null){
                sq_qg_assign_json_str = sq_qg_assign_json_str.trim()
            }
            if(sq_qg_assign_json_str != null && !sq_qg_assign_json_str.isEmpty()){
                echo "No se pudo asignar el Umbral de Calidad "+
                "con id \'${sq_qg_id}\' al Proyecto con id \'${sq_project_key}\'. "+
                "salio el siguiente mensaje: \'${sq_qg_assign_json_str}\', por favor verificar"
                error_during_assignment = true;
            }
          }
        }
	  }
	  result.error = error_during_assignment
	  now = new Date().format('yyyy-MM-dd_HH.mm.ss')
      echo """====================================================================================================
FIN DE CREACION DE PROYECTO SONARQUBE ${now}
===================================================================================================="""
      return result
    }
  }
}

def deleteSonarQubeProject(sonarqube_server_name, jenkins_sonarqube_credential_id, scan_prep_info, production_branch){
  withSonarQubeEnv(sonarqube_server_name){
	def sq_url = env.SONAR_HOST_URL
    def project_deletion_url = "${sq_url}/api/projects/delete?project=${scan_prep_info.projectKey}"
	def sq_project_deletion_request = httpRequest httpMode: 'POST', authentication:"${jenkins_sonarqube_credential_id}", acceptType: 'APPLICATION_JSON_UTF8', url: project_deletion_url
    if(sq_project_deletion_request.status == 204){
      echo "se borro el proyecto con con llave '${scan_prep_info.projectKey}'"
    } else {
      echo "El proyecto con llave '${scan_prep_info.projectKey}' no se borro satisfactoriamente, por favor verificar"
      return false;
	}
	return true;
  }
}
def compileStep(maven_pom_file_path, build_workspace, jdk_tool_name, maven_tool_name, maven_compilation_flags, maven_goals){
	def now = new Date().format('yyyy-MM-dd_HH.mm.ss')
echo """====================================================================================================
INICIO DE COMPILACION DE CODIGO ${now}
===================================================================================================="""
  def maven_pom_file_full_path_parameter = (maven_pom_file_path != null && !maven_pom_file_path.isEmpty()) ? "-f '${build_workspace}/${maven_pom_file_path}'" : ""
  jdk = tool name: jdk_tool_name
  maven = tool name: maven_tool_name
  env.JAVA_HOME = "${jdk}"
  env.M2_HOME = "${maven}"
  sh "'$M2_HOME'/bin/mvn ${maven_compilation_flags} ${maven_pom_file_full_path_parameter} ${maven_goals}"
  now = new Date().format('yyyy-MM-dd_HH.mm.ss')
echo """====================================================================================================
FIN DE COMPILACION DE CODIGO ${now}
===================================================================================================="""
}

def codeScanStep(sonarqube_server_name, maven_pom_file_path, build_workspace, jdk_tool_name, maven_tool_name, maven_compilation_flags, maven_goals, maven_sonarqube_scan_plugin_name_and_version, 
  sq_project_info, git_repo_info, sonar_pom_properties_override, amount_of_minutes_to_wait_for_scan_to_finish, stop_build_on_quality_gate_failure) {
  prepProyectForSonarQubeScan(maven_pom_file_path, build_workspace, jdk_tool_name, maven_tool_name, sq_project_info, git_repo_info, sonar_pom_properties_override)
  def now = new Date().format('yyyy-MM-dd_HH.mm.ss')
echo """====================================================================================================
INICIO DE COMPILACION Y ESCANEO DE CODIGO ${now}
===================================================================================================="""
  dir (build_workspace){
	withSonarQubeEnv(sonarqube_server_name) {
	  def maven_pom_file_full_path_parameter = (maven_pom_file_path != null && !maven_pom_file_path.isEmpty()) ? "-f '${build_workspace}/${maven_pom_file_path}'" : ""
	  jdk = tool name: jdk_tool_name
      maven = tool name: maven_tool_name
      env.JAVA_HOME = "${jdk}"
      env.M2_HOME = "${maven}"
      sh "'$M2_HOME'/bin/mvn ${maven_compilation_flags} ${maven_pom_file_full_path_parameter} ${maven_goals} ${maven_sonarqube_scan_plugin_name_and_version}:sonar"
    }
	// this sleep(10) is needed to avoid the waitForQualityGate looping endlessly
	sleep(10)
    timeout(amount_of_minutes_to_wait_for_scan_to_finish) { 
      def quality_gate = waitForQualityGate();
      if (quality_gate.status != 'OK') {
		if (stop_build_on_quality_gate_failure){
		  error "Despliegue se detuvo por falla en el umbral de calidad: ${quality_gate.status}"	
		} else {
		  echo "status fallido del umbral de calidad: ${quality_gate.status}, pero no se para el despliegue"
		}
      } else{
		echo "status exitoso del umbral de calidad: ${quality_gate.status}"
	  }
    }	
  }
now = new Date().format('yyyy-MM-dd_HH.mm.ss')
echo """====================================================================================================
FIN DE COMPILACION Y ESCANEO DE CODIGO ${now}
===================================================================================================="""
}

def uninstallApplicationsStep(app_name_to_deploy_configs, app_names_to_deploy, module_server_mapping_key_to_configs, topology_key_to_configs, dmgr_server_key_to_configs,
  key_value_sep, key_value_pair_sep, key_value_pair_groups_sep){
  def apps_to_uninstall = app_name_to_deploy_configs.findAll{entry -> app_names_to_deploy.contains(entry.key)}
  def dmgr_server_keys_to_apps_to_uninstall = [:]
  apps_to_uninstall.each { app_to_install_name, deploy_configurations ->
    deploy_configurations.each{ deploy_configuration ->
      def dmgr_server_key = deploy_configuration.get('dmgr_server_key')
      //app_deploy_config = ['ear_name':ear_name, 'installation_type':installation_type, 'dmgr_server_key':dmgr_server_key, 'topology_key': topology_key]
      addItemToCollectionInMapValue(deploy_configuration, dmgr_server_key, dmgr_server_keys_to_apps_to_uninstall, [])
    }
  }
  if(dmgr_server_keys_to_apps_to_uninstall.isEmpty()){
    echo "no hay ninguna aplicacion configurada para desinstalar"
    return
  }
  echo "dmgr_server_keys_to_apps_to_uninstall ${dmgr_server_keys_to_apps_to_uninstall}"
  def uninstall_config_closures = [0 : [{ map, kv_sep -> "nombre_app${kv_sep}${map.app_name}"}], 
    1 : [{ map, kv_sep, kv_pair_sep -> "servidor_app${kv_sep}${map.app_server}${kv_pair_sep}nodo_app${kv_sep}${map.app_node}${kv_pair_sep}celda_app${kv_sep}${map.app_cell}"}] ]
  dmgr_server_keys_to_apps_to_uninstall.each{ dmgr_server_key, deploy_configurations ->
    def apps_with_topologies_to_uninstall = []
    deploy_configurations.each { deploy_configuration ->
	  echo "deploy_configuration ${deploy_configuration}"
      module_server_mapping_key_to_configs.get(deploy_configuration.get('module_server_mapping_key')).each{ module_server_mapping_config ->
	    echo "module_server_mapping_config ${module_server_mapping_config}"
        topology_key_to_configs.get(module_server_mapping_config.get('topology_key')).each{ topology_config ->
		  echo "topology_config ${topology_config}"
          apps_with_topologies_to_uninstall.add(joinObjectsIntoConfigurationString ([deploy_configuration, topology_config], 
            uninstall_config_closures, key_value_sep, key_value_pair_sep))
        }
      }
    }
    def apps_with_topologies_to_uninstall_str = apps_with_topologies_to_uninstall.join(key_value_pair_groups_sep)
    echo "apps_with_topologies_to_uninstall_str $apps_with_topologies_to_uninstall_str"
    def dmgr_server_configs = dmgr_server_key_to_configs.get(dmgr_server_key)
    dmgr_server_configs.each{ dmgr_server_config -> 
	  def jenkins_server_and_dmgr_server_are_the_same = ("true".equals(dmgr_server_config.jenkins_server_and_dmgr_server_are_the_same))
      def uninstall_sh_cmd = "'${dmgr_server_config.dmgr_deploy_scripts_directory}desinstalacionDeEARS.sh' "+
           "-dmgrProfilePath '${dmgr_server_config.dmgr_profile_path}' "+
           "-wsadminPropertiesFile '${dmgr_server_config.wsadmin_credential_properties_file_path}' "+
           "-appNamesAndTopologies '${apps_with_topologies_to_uninstall_str}' "
	  executeShellCommandOnServer(uninstall_sh_cmd, dmgr_server_config.server_credentials_id, dmgr_server_config.server_ip, dmgr_server_config.ssh_server_port, jenkins_server_and_dmgr_server_are_the_same, 
	    'Error al ejecutar el script para desinstalar los EARS del WAS, por favor revise los logs')
    }
  }
}

def jarReplacementStep (buildable_jar_name_to_shared_library_keys, jar_names_to_replace, app_name_to_shared_library_keys, app_name_to_deploy_configs, app_names_to_deploy,
  shared_library_key_to_configs, shared_library_server_key_to_configs, build_workspace, dmgr_server_key_to_configs, module_server_mapping_key_to_configs, topology_key_to_configs, 
  jars_to_replace_base_directory, key_value_sep, key_value_pair_sep, key_value_pair_groups_sep){
  def buildable_jar_name_to_shared_library_keys_to_replace = buildable_jar_name_to_shared_library_keys.findAll{entry -> jar_names_to_replace.contains(entry.key)}
  if(buildable_jar_name_to_shared_library_keys_to_replace.isEmpty()){
    echo "no hay ningun jar configurado para reemplazar"
    return
  }
  // First the applications that use the shared libraries that have not been uninstalled must be stopped
  def shared_library_key_to_app_names = [:]
  app_name_to_shared_library_keys.each { app_name, app_name_and_shared_library_keys ->
    app_name_and_shared_library_keys.each { app_name_and_shared_library_key ->
      def shared_library_key = app_name_and_shared_library_key.get('shared_library_key')
      addItemToCollectionInMapValue(app_name, shared_library_key, shared_library_key_to_app_names, [].toSet())
    }
  }
  
  def app_names_to_restart = [].toSet()
  def shared_library_server_key_to_shared_library_keys = [:]
  def dmgr_server_key_to_shared_library_keys_to_replace = [:]
  buildable_jar_name_to_shared_library_keys_to_replace.each { jar_name, shared_library_keys_and_jar_name ->
    shared_library_keys_and_jar_name.each { shared_library_key_and_jar_name ->
      def shared_library_key  = shared_library_key_and_jar_name.get('shared_library_key')
      shared_library_key_to_configs.get(shared_library_key).each {shared_library_config ->
        def shared_library_server_key = shared_library_config.get('shared_library_server_key')
		def dmgr_server_key = shared_library_config.get('dmgr_server_key')
		addItemToCollectionInMapValue(shared_library_key, dmgr_server_key, dmgr_server_key_to_shared_library_keys_to_replace, [].toSet())
		addItemToCollectionInMapValue(shared_library_key, shared_library_server_key, shared_library_server_key_to_shared_library_keys, [].toSet())	
      }
      app_names_to_restart.addAll(shared_library_key_to_app_names.get(shared_library_key))
    }
  }
  app_names_to_restart.removeAll(app_names_to_deploy)
  def apps_to_restart = app_name_to_deploy_configs.findAll{entry -> app_names_to_restart.contains(entry.key)}
  echo "app_names_to_restart ${app_names_to_restart}"
  echo "apps_to_restart ${apps_to_restart}"
  echo "shared_library_server_key_to_shared_library_keys ${shared_library_server_key_to_shared_library_keys}"
  echo "dmgr_server_key_to_shared_library_keys_to_replace ${dmgr_server_key_to_shared_library_keys_to_replace}"
  
  def restart_config_closures = [0 : [{ map, sep -> "nombre_app${sep}${map.app_name}"}], 
    1 : [{ map, kv_sep, kv_pair_sep -> "mapeo_gestor_aplicaciones${kv_sep}cell=${map.app_cell},node=${map.app_node},type=ApplicationManager,process=${map.app_server},*"}] ]
  def dmgr_server_keys_to_apps_with_topologies_to_restart = [:]
  def dmgr_server_keys_to_apps_with_topology_config_str = [:]
  apps_to_restart.each { app_to_install_name, deploy_configurations ->
    deploy_configurations.each { deploy_configuration -> 
      def configuration_of_app_to_restart = []
      def dmgr_server_key = deploy_configuration.get('dmgr_server_key')
      dmgr_server_key_to_configs.get(dmgr_server_key).each{ dmgr_server_config ->
        module_server_mapping_key_to_configs.get(deploy_configuration.get('module_server_mapping_key')).each{ module_server_mapping_config ->
          topology_key_to_configs.get(module_server_mapping_config.get('topology_key')).each{ topology_config ->
            configuration_of_app_to_restart.add(joinObjectsIntoConfigurationString ([deploy_configuration, topology_config], 
            restart_config_closures, key_value_sep, key_value_pair_sep))
          }
        }
      }
      addItemToCollectionInMapValue(configuration_of_app_to_restart.join(key_value_pair_groups_sep), dmgr_server_key, dmgr_server_keys_to_apps_with_topologies_to_restart, [])
    }
  }
  echo "dmgr_server_keys_to_apps_with_topologies_to_restart ${dmgr_server_keys_to_apps_with_topologies_to_restart}"
  dmgr_server_keys_to_apps_with_topologies_to_restart.each{ dmgr_server_key, apps_with_topologies_to_stop ->
    def apps_with_topologies_to_stop_config_str = apps_with_topologies_to_stop.join(key_value_pair_groups_sep)
    echo "apps_with_topologies_to_stop_config_str: ${apps_with_topologies_to_stop_config_str}"
    dmgr_server_key_to_configs.get(dmgr_server_key).each{ dmgr_server_config ->
	  def jenkins_server_and_dmgr_server_are_the_same = ("true".equals(dmgr_server_config.jenkins_server_and_dmgr_server_are_the_same))
      def stop_sh_cmd = "'${dmgr_server_config.dmgr_deploy_scripts_directory}detenimientoDeAplicaciones.sh' " +
           "-dmgrProfilePath '${dmgr_server_config.dmgr_profile_path}' "+
           "-wsadminPropertiesFile '${dmgr_server_config.wsadmin_credential_properties_file_path}' "+
           "-appNamesAndTopologiesToStop '${apps_with_topologies_to_stop_config_str}' "
      executeShellCommandOnServer(stop_sh_cmd, dmgr_server_config.server_credentials_id, dmgr_server_config.server_ip, dmgr_server_config.ssh_server_port, jenkins_server_and_dmgr_server_are_the_same, 
      'Error al ejecutar el script de detencion de aplicaciones que no se desplegaran, por favor revise los logs')
    }
  }
  // then the jars must be replaced
  def shared_library_key_and_shared_library_server_key_to_cached_path_on_server = [:]
  def shared_library_query_closures = [0 : [{ map, kv_sep, kv_pair_sep ->  "nombre_libreria_compartida${kv_sep}${map.shared_library_name}${kv_pair_sep}nombre_objeto_contenedor${kv_sep}${map.container_obj_name}${kv_pair_sep}tipo_objeto_contenedor${kv_sep}${map.container_obj_type}"}],
    1 : [{ value, kv_sep -> "llave_libreria_a_imprimir${kv_sep}${value}"}] ]
  dmgr_server_key_to_shared_library_keys_to_replace.each { dmgr_server_key, shared_library_keys ->
    dmgr_server_key_to_configs.get(dmgr_server_key).each{dmgr_server_config ->
      def shared_library_configs_to_query = []
      shared_library_keys.each{ shared_library_key ->
        shared_library_key_to_configs.get(shared_library_key).findAll{config -> dmgr_server_key.equals(config.dmgr_server_key)}.each{shared_library_config ->
          def shared_library_config_str = joinObjectsIntoConfigurationString([shared_library_config, shared_library_key], shared_library_query_closures, key_value_sep, key_value_pair_sep)
          shared_library_configs_to_query.add("${shared_library_config_str}")
        }
      }
      def shared_library_configs_to_query_str = shared_library_configs_to_query.join(key_value_pair_groups_sep)
      def shared_library_sh_cmd = "'${dmgr_server_config.dmgr_deploy_scripts_directory}consultarLibreriasCompartidas.sh' " +
           "-dmgrProfilePath '${dmgr_server_config.dmgr_profile_path}' "+
           "-wsadminPropertiesFile '${dmgr_server_config.wsadmin_credential_properties_file_path}' "+
           "-sharedLibraryConfigsToQuery '${shared_library_configs_to_query_str}' "
      def shared_library_paths_config_str = ''
      if("true".equals(dmgr_server_config.jenkins_server_and_dmgr_server_are_the_same)){
        shared_library_paths_config_str = sh returnStdout: true, script: shared_library_sh_cmd
      }else{
        withCredentials([usernamePassword(credentialsId: dmgr_server_config.server_credentials_id, passwordVariable: 'password', usernameVariable: 'user')]) {
          def remote = ['name':dmgr_server_key, 'host':dmgr_server_config.server_ip, 'port':dmgr_server_config.ssh_server_port.toInteger(), 'user':user, 'password':password, 'allowAnyHosts':true]
          shared_library_paths_config_str = sshCommand remote: remote, failOnError: true, command: shared_library_sh_cmd
        }
      }
      echo "${shared_library_paths_config_str}"
      def shared_library_paths_config_strings = shared_library_paths_config_str.split('\n');
      shared_library_paths_config_strings.eachWithIndex{ shared_library_path_config_str, idx -> 
        if (idx >= shared_library_paths_config_strings.size() - shared_library_configs_to_query.size()){
          def shared_library_path_config= shared_library_path_config_str.split('\u0001');
  	      if(shared_library_path_config.length > 1){ 
			shared_library_key_to_configs.get(shared_library_path_config[0]).findAll{config -> dmgr_server_key.equals(config.dmgr_server_key)}.each{ shared_library_config ->
			  shared_library_key_and_shared_library_server_key_to_cached_path_on_server.put(
			  "${shared_library_path_config[0]}_${shared_library_config.shared_library_server_key}", shared_library_path_config[1].replaceAll('\r',''));
			}
  	      }
        }
      }
    }
  }
  
  echo "shared_library_key_and_shared_library_server_key_to_cached_path_on_server ${shared_library_key_and_shared_library_server_key_to_cached_path_on_server}"
  def now = new Date().format('yyyy-MM-dd_HH.mm.ss')
  echo """====================================================================================================
INICIO DE REEMPLAZO DE JARS DE LIBRERIAS COMPARTIDAS ${now}"
===================================================================================================="""
  def shared_library_key_to_jar_paths_to_move = [:]
  dir (build_workspace) {
    buildable_jar_name_to_shared_library_keys_to_replace.each { buildable_jar_name, shared_library_keys_and_jar_name ->
      def jar_files = findFiles glob: "**/${buildable_jar_name}"
      shared_library_keys_and_jar_name.each { shared_library_key_and_jar_name ->
        def shared_library_key  = shared_library_key_and_jar_name.get('shared_library_key')
        addItemToCollectionInMapValue("'"+jar_files[0].path+"'", shared_library_key, shared_library_key_to_jar_paths_to_move, [].toSet())
      }
    }
  }
  shared_library_server_key_to_shared_library_keys.each{ shared_library_server_key, shared_library_keys ->
    shared_library_keys.each{ shared_library_key ->
      def jar_paths_to_move = shared_library_key_to_jar_paths_to_move.get(shared_library_key)
      def paths_to_copy = jar_paths_to_move.join(' ');
      def sh_status = sh returnStatus: true, script: "cd '${build_workspace}/'; mkdir -p '${jars_to_replace_base_directory}${shared_library_server_key}/${shared_library_key}/'; cp -f ${paths_to_copy} '${jars_to_replace_base_directory}${shared_library_server_key}/${shared_library_key}/';"
      if (sh_status != 0){
        error('Error al ejecutar el script para mover los Jars de las carpetas de las fuentes a las carpetas internas del Jenkins, por favor revise los logs')
      }
    }
  }
  shared_library_server_key_to_shared_library_keys.each{ shared_library_server_key, shared_library_keys ->
    shared_library_server_key_to_configs.get(shared_library_server_key).each{ shared_library_server_config ->
	  def jenkins_server_and_library_server_are_the_same = ("true".equals(shared_library_server_config.jenkins_server_and_library_server_are_the_same))
	  def artifact_directory = shared_library_server_config.get('server_artifact_directory')
      if(jenkins_server_and_library_server_are_the_same){
        echo "no es necesario mover los jars a otro servidor con llave ${shared_library_server_key} antes de reemplazarlos en las librerias compartidas"
      }else{
        def server_credentials_id = shared_library_server_config.get('server_credentials_id')
        withCredentials([usernamePassword(credentialsId: server_credentials_id, passwordVariable: 'password', usernameVariable: 'user')]) {
          def remote = ['name':shared_library_server_config.server_ip, 'host':shared_library_server_config.server_ip, 'port':shared_library_server_config.ssh_server_port.toInteger(), 'user':user, 'password':password, 'allowAnyHosts':true]
  	      sshPut remote: remote, from: "${build_workspace}/${jars_to_replace_base_directory}${shared_library_server_key}/", into: "${artifact_directory}"
        }
      }
      def move_sh_cmd = ''
      shared_library_keys.each{ shared_library_key -> 
		def shared_library_path_on_server = shared_library_key_and_shared_library_server_key_to_cached_path_on_server.get("${shared_library_key}_${shared_library_server_key}")
        def base_directory_of_jars_to_replace = (jenkins_server_and_library_server_are_the_same) ? "${build_workspace}/${jars_to_replace_base_directory}": artifact_directory;
        move_sh_cmd += "mv -f '${base_directory_of_jars_to_replace}${shared_library_server_key}/${shared_library_key}/'* '${shared_library_path_on_server}' ; "
      }
	  echo "move_sh_cmd ${move_sh_cmd}"
      executeShellCommandOnServer(move_sh_cmd, shared_library_server_config.server_credentials_id, shared_library_server_config.server_ip, shared_library_server_config.ssh_server_port, jenkins_server_and_library_server_are_the_same, 
        'Error al ejecutar el script para mover los Jars de las carpetas internas del Jenkins a las carpetas de las librerias compartidas del WAS, por favor revise los logs')
    }
  }
  now = new Date().format('yyyy-MM-dd_HH.mm.ss')
  echo """====================================================================================================
FIN DE REEMPLAZO DE JARS DE LIBRERIAS COMPARTIDAS ${now}
===================================================================================================="""
  // and finally the apps that use the shared libraries that have not been uninstalled must be started again
  dmgr_server_keys_to_apps_with_topologies_to_restart.each{ dmgr_server_key, apps_with_topologies_to_start ->
    def apps_with_topologies_to_start_config_str = apps_with_topologies_to_start.join(key_value_pair_groups_sep)
    echo "apps_with_topologies_to_start_config_str: ${apps_with_topologies_to_start_config_str}"
    dmgr_server_key_to_configs.get(dmgr_server_key).each{ dmgr_server_config ->
	  def jenkins_server_and_dmgr_server_are_the_same = ("true".equals(dmgr_server_config.jenkins_server_and_dmgr_server_are_the_same))
      def start_sh_cmd = "'${dmgr_server_config.dmgr_deploy_scripts_directory}inicioDeAplicaciones.sh' " +
           "-dmgrProfilePath '${dmgr_server_config.dmgr_profile_path}' "+
           "-wsadminPropertiesFile '${dmgr_server_config.wsadmin_credential_properties_file_path}' "+
           "-appNamesAndTopologiesToStart '${apps_with_topologies_to_start_config_str}' "
      executeShellCommandOnServer(start_sh_cmd, dmgr_server_config.server_credentials_id, dmgr_server_config.server_ip, dmgr_server_config.ssh_server_port, jenkins_server_and_dmgr_server_are_the_same, 
        'Error al ejecutar el script de inicio de aplicaciones que no se desplegaran, por favor revise los logs')
    }
  }
}

def deployApplicationsStep(app_name_to_deploy_configs, app_names_to_deploy, build_workspace, ears_to_deploy_directory, dmgr_server_key_to_configs, module_server_mapping_key_to_configs, 
  topology_key_to_configs, role_to_user_and_or_group_mapping_key_to_configs, session_management_config_key_to_configs, tuning_parameters_key_to_configs, 
  cookie_config_key_to_configs, app_name_to_shared_library_keys, shared_library_key_to_configs, amount_of_times_to_check_app_status_to_start, 
  interval_between_checks_of_app_status_to_start_in_milliseconds, tcl_empty_string, key_value_sep, key_value_pair_sep, key_value_pair_groups_sep, 
  internal_config_key_value_sep, internal_config_key_value_pair_sep){
  def apps_to_install = app_name_to_deploy_configs.findAll{entry -> app_names_to_deploy.contains(entry.key)}
  def dmgr_server_keys_to_apps_to_install = [:]
  apps_to_install.each { app_to_install_name, deploy_configurations ->
    deploy_configurations.each { deploy_configuration -> 
      def dmgr_server_key = deploy_configuration.get('dmgr_server_key')
      addItemToCollectionInMapValue(deploy_configuration, dmgr_server_key, dmgr_server_keys_to_apps_to_install, [])
    }
  }
  echo "dmgr_server_keys_to_apps_to_install: ${dmgr_server_keys_to_apps_to_install}"
  def are_there_apps_to_install = !dmgr_server_keys_to_apps_to_install.isEmpty()
  if(!are_there_apps_to_install){
    echo "no hay ninguna aplicacion configurada para desplegar"
    return
  }
  def ear_paths_to_move = [].toSet()
  dir (build_workspace) {
    apps_to_install.each { app_name, app_configs ->
      app_configs.each { app_config ->
        def ear_files = findFiles glob: "**/${app_config.ear_name}"
        ear_paths_to_move.add("'"+ear_files[0].path+"'")
      }
    }
  }
  def paths_to_copy = ear_paths_to_move.join(' ')
  def cp_sh_status = sh returnStatus: true, script: "cd '${build_workspace}/'; mkdir -p '${ears_to_deploy_directory}'; cp -f ${paths_to_copy} '${ears_to_deploy_directory}'"
  if (cp_sh_status != 0){
    error('Error al ejecutar el script para mover los EARS de las fuentes a las carpetas internas del Jenkins para el despliegue, por favor revise los logs')
  }

  def app_install_closures = [0 : [{ value, kv_sep -> "nombre_app${kv_sep}${value}"}], 1 : [{ value, kv_sep -> "ruta_ear${kv_sep}${value}"}], 
	2 : [{list, kv_sep, kv_pair_sep -> (list) ? list.collect{item -> "mapeo_modulo_a_servidor${kv_sep}${item}"}.join(kv_pair_sep) : ''} ],
	3 : [{list, kv_sep, kv_pair_sep -> (list) ? list.collect{item -> "mapeo_gestor_aplicaciones${kv_sep}${item}"}.join(kv_pair_sep) : ''} ],
	4 : [{list, kv_sep, kv_pair_sep -> (list) ? list.collect{item -> "mapeo_rol_a_usuario_y_o_grupo${kv_sep}${item}"}.join(kv_pair_sep) : ''} ],
	5 : [{str, kv_sep, kv_pair_sep -> (str) ? "mapeo_gestor_sesion${kv_sep}${str}" : ''} ],
	6 : [{str, kv_sep, kv_pair_sep -> (str) ? "mapeo_parametros_afinacion${kv_sep}${str}" : ''} ],
	7 : [{str, kv_sep, kv_pair_sep -> (str) ? "mapeo_parametros_cookie${kv_sep}${str}" : ''} ],
    8 : [{list, kv_sep, kv_pair_sep -> (list) ? list.collect{item -> "libreria_compartida${kv_sep}${item}"}.join(kv_pair_sep) : ''} ],
	9 : [{list, kv_sep, kv_pair_sep -> (list) ? list.collect{item -> "nodo_app${kv_sep}${item}"}.join(kv_pair_sep) : ''} ],
	10 : [{list, kv_sep, kv_pair_sep -> (list) ? list.collect{item -> "celda_app${kv_sep}${item}"}.join(kv_pair_sep) : ''} ]]
  dmgr_server_keys_to_apps_to_install.each{ dmgr_server_key, apps_to_install_configs ->
    def install_configurations_of_apps = []
    apps_to_install_configs.each { deploy_configuration ->
      def app_name = deploy_configuration.get('app_name')
      dmgr_server_key_to_configs.get(dmgr_server_key).each{ dmgr_server_config ->
		def module_server_mappings = []
		def application_manager_mappings = []
		def distinct_deploy_nodes = [].toSet()
		def distinct_deploy_cells = [].toSet()
        module_server_mapping_key_to_configs.get(deploy_configuration.get('module_server_mapping_key')).each{ module_server_mapping_config ->
          topology_key_to_configs.get(module_server_mapping_config.get('topology_key')).each{ topology_config ->
			def deploy_server_mapping = (topology_config.app_cluster) ?
			  "WebSphere:cell=${topology_config.app_cell},cluster=${topology_config.app_cluster}" : "WebSphere:cell=${topology_config.app_cell},node=${topology_config.app_node},server=${topology_config.app_server}";
			application_manager_mappings.add("cell=${topology_config.app_cell},node=${topology_config.app_node},type=ApplicationManager,process=${topology_config.app_server},*");
			def ihs_server_mapping = ( topology_config.ihs_server && topology_config.ihs_node  && topology_config.ihs_cell ) ?
			  "+WebSphere:cell=${topology_config.ihs_cell},node=${topology_config.ihs_node},server=${topology_config.ihs_server}" : '';
			module_server_mappings.add("{{${module_server_mapping_config.config_file_display_name}} {${module_server_mapping_config.artifact_and_configuration_file_path}} {${deploy_server_mapping}${ihs_server_mapping}}}");
			distinct_deploy_nodes.add(topology_config.app_node);
			distinct_deploy_cells.add(topology_config.app_cell);
          }
        }
		
		def role_to_user_and_or_group_mappings = []
        role_to_user_and_or_group_mapping_key_to_configs.get(deploy_configuration.get('role_to_user_and_or_group_mapping_key')).each{ config ->
		  def user_mapping = (config.users_to_map) ? config.users_to_map  : "${tcl_empty_string}"
		  def group_mapping = (config.groups_to_map) ? config.groups_to_map : "${tcl_empty_string}"
		  def allow_all_authenticated_users_in_application_domain = config.allow_all_authenticated_users_in_application_domain
		  role_to_user_and_or_group_mappings.add("{${config.role_name} ${config.allow_access_to_all} ${config.allow_access_to_authenticated_users} ${user_mapping} ${group_mapping} ${allow_all_authenticated_users_in_application_domain}}");
        }

        echo "session_management_config_key_to_configs.values().flatten() ${session_management_config_key_to_configs.values().flatten()}"
		def session_management_configs = session_management_config_key_to_configs.get(deploy_configuration.get('session_management_config_key'))
		def session_management_config_str = ''
		def tuning_parameters_config_str = ''
		def cookie_config_str = ''
		if (session_management_configs){
		  def session_management_config = session_management_configs.flatten().first()
		  session_management_config_str = session_management_config.findAll{!['session_management_config_key','cookie_config_key','tuning_parameters_key'].contains(it.key)}
		  .collect([]){"${it.key}${internal_config_key_value_sep}${it.value}"}.join(internal_config_key_value_pair_sep)
		  tuning_parameters_config_str = tuning_parameters_key_to_configs.get(session_management_config.get('tuning_parameters_key')).flatten().first().findAll{!'tuning_parameters_key'.equals(it.key)}
		  .collect([]){"${it.key}${internal_config_key_value_sep}${it.value}"}.join(internal_config_key_value_pair_sep)
		  cookie_config_str = cookie_config_key_to_configs.get(session_management_config.get('cookie_config_key')).flatten().first().findAll{!'cookie_config_key'.equals(it.key)}
		  .collect([]){"${it.key}${internal_config_key_value_sep}${it.value}"}.join(internal_config_key_value_pair_sep) 			
		}

		def shared_library_names = app_name_to_shared_library_keys.get(app_name).collect([]){shared_library_key_to_configs.get(it.shared_library_key)}
		.flatten().collect([].toSet()){it.shared_library_name}
        def dmgr_artifact_directory = dmgr_server_config.get('dmgr_artifact_directory');
        def ear_path = ("true".equals(dmgr_server_config.jenkins_server_and_dmgr_server_are_the_same)) ? 
          "${build_workspace}/${ears_to_deploy_directory}" + deploy_configuration.get('ear_name') : "${dmgr_artifact_directory}" + deploy_configuration.get('ear_name');
        install_configurations_of_apps.add(joinObjectsIntoConfigurationString([app_name, ear_path, module_server_mappings, application_manager_mappings, role_to_user_and_or_group_mappings,
          session_management_config_str, tuning_parameters_config_str, cookie_config_str, shared_library_names, distinct_deploy_nodes, distinct_deploy_cells],
          app_install_closures, key_value_sep, key_value_pair_sep))
      }
    }
    def install_configurations_of_apps_str = install_configurations_of_apps.join(key_value_pair_groups_sep)
    echo "install_configurations_of_apps_str: ${install_configurations_of_apps_str}"
    dmgr_server_key_to_configs.get(dmgr_server_key).each{ dmgr_server_config ->
      def deploy_sh_cmd = "'${dmgr_server_config.dmgr_deploy_scripts_directory}despliegueDeEARS.sh' " +
           "-dmgrProfilePath '${dmgr_server_config.dmgr_profile_path}' "+
           "-wsadminPropertiesFile '${dmgr_server_config.wsadmin_credential_properties_file_path}' "+
           "-amountOfTimesToQueryAppState ${amount_of_times_to_check_app_status_to_start} " + 
           "-appStateQueryIntervalInMilliseconds ${interval_between_checks_of_app_status_to_start_in_milliseconds} " +
           "-installConfigurationsOfApps '${install_configurations_of_apps_str}' "
      if("true".equals(dmgr_server_config.jenkins_server_and_dmgr_server_are_the_same)){
        def sh_status = sh returnStatus: true, script: deploy_sh_cmd
        echo " sh_status ${sh_status}"
        if (sh_status != 0){
          error('Error al ejecutar el script de despliegue de EARS, por favor revise los logs')
        }
      }else{
        withCredentials([usernamePassword(credentialsId: dmgr_server_config.server_credentials_id, passwordVariable: 'password', usernameVariable: 'user')]) {
          def remote = ['name':dmgr_server_key, 'host':dmgr_server_config.server_ip, 'port':dmgr_server_config.ssh_server_port.toInteger(), 'user':user, 'password':password, 'allowAnyHosts':true]
		  def dmgr_artifact_directory = dmgr_server_config.get('dmgr_artifact_directory');
          sshPut remote: remote, failOnError :true, from: "${build_workspace}/${ears_to_deploy_directory}", into: "${dmgr_artifact_directory}"
          sshCommand remote: remote, failOnError :true ,command: "mv -f '${dmgr_artifact_directory}${ears_to_deploy_directory}'*.ear '${dmgr_artifact_directory}'; rm -rf '${dmgr_artifact_directory}${ears_to_deploy_directory}'"
          sshCommand remote: remote, failOnError :true ,command: deploy_sh_cmd
        }
      }
    }
  }
}

def artifactoryUploadStep(build_workspace, artifactory_server_id, artifactory_relative_directory_path_to_upload_ears, app_name_to_deploy_configs, app_names_to_deploy, 
  buildable_jar_name_to_shared_library_keys, jar_names_to_replace, artifactory_relative_directory_path_to_upload_jars, upload_all_configured_artifacts_in_properties_file_to_artifactory){
  dir (build_workspace){
    def now_timestamp = new java.text.SimpleDateFormat('YYYY_MM_dd_HH_mm_ss').format(new java.util.Date())
    def artifactory_server = Artifactory.server artifactory_server_id
    // Create the upload spec, which is in json format
    def upload_spec = """
    { "files": 
	  ["""
	def buildable_jar_name_to_shared_library_keys_to_upload = [:]
	def apps_to_upload = [:]
	def first_spec_added = false
	if(upload_all_configured_artifacts_in_properties_file_to_artifactory){
      apps_to_upload = app_name_to_deploy_configs
	  buildable_jar_name_to_shared_library_keys_to_upload = buildable_jar_name_to_shared_library_keys
	}else{
      apps_to_upload = app_name_to_deploy_configs.findAll{entry -> app_names_to_deploy.contains(entry.key)}
	  buildable_jar_name_to_shared_library_keys_to_upload = buildable_jar_name_to_shared_library_keys.findAll{entry -> jar_names_to_replace.contains(entry.key)}
	}
	apps_to_upload.each { app_name, app_configs ->
      app_configs.each { app_config ->
        def ear_files = findFiles glob: "**/${app_config.ear_name}"
		def spec_separator = (first_spec_added) ? ',' : '';
		upload_spec += """
		${spec_separator}{
            "pattern": "${ear_files[0].path}",
            "target": "${artifactory_relative_directory_path_to_upload_ears}${now_timestamp}/"
        }"""
		first_spec_added = (first_spec_added || true)
      }
    }
    buildable_jar_name_to_shared_library_keys_to_upload.each { buildable_jar_name, shared_library_keys_and_jar_name ->
      def jar_files = findFiles glob: "**/${buildable_jar_name}"
	  def spec_separator = (first_spec_added) ? ',' : '';
		upload_spec += """
        ${spec_separator}{
           "pattern": "${jar_files[0].path}",
           "target": "${artifactory_relative_directory_path_to_upload_jars}${now_timestamp}/"
        }"""
		first_spec_added = (first_spec_added || true)
    }
    upload_spec+="""
	  ]
    }"""
    echo "upload_spec: ${upload_spec}"
    echo "artifactory_server ${artifactory_server}"
    def upload_info = artifactory_server.upload spec: upload_spec
    echo "info de artifactory ${upload_info}"
  }
}

enum INITIAL_SCAN_TYPES {
  NONE,
  NO_SOURCES,
  PRODUCTION
  public INITIAL_SCAN_TYPES(){}
}
node{
    properties(
        [
            parameters([
                string(name: 'RUTA_DE_REPO_DE_PARCHE', defaultValue: params.RUTA_DE_REPO_DE_PARCHE, description: 'La URL del repositorio de parches, por favor asegurese que apunte al repositorio correcto', trim: false),
                string(name: 'ID_DE_CREDENCIALES_DE_REPO_DE_PARCHE', defaultValue: params.ID_DE_CREDENCIALES_DE_REPO_DE_PARCHE, description: 'El id de los credenciales configurados en jenkins para acceder al repositorio de parches, verifique que apunte a un id de credenciales existente', trim: false),
                string(name: 'BRANCH_DEL_REPO_DE_PARCHE_A_USAR', defaultValue: params.BRANCH_DEL_REPO_DE_PARCHE_A_USAR, description: 'Rama del repositorio de parches que contiene el archivo de propiedades para realizar el despliegue de este pipeline', trim: false),
                string(name: 'RUTA_DEL_ARCHIVO_DE_CONFIGURACION_EN_EL_REPO_DE_PARCHE', defaultValue: params.RUTA_DEL_ARCHIVO_DE_CONFIGURACION_EN_EL_REPO_DE_PARCHE, description: 'Ruta del archivo que contiene las propiedades para realizar el despliegue de este pipeline', trim: false),
                text(name: 'NOMBRES_DE_APLICACIONES_A_DESPLEGAR', defaultValue:  '''cavali-fn-client-web-app-ear
cavali-fn-web-services-client-backend-ear
cavali-fn-web-services-core-backend-ear''', description: 'Los nombres de las aplicaciones a desplegar, cada renglon tendra una aplicacion a desplegar', trim: false),
                text(name: 'JARS_A_REEMPLAZAR', defaultValue: '''cavali-fn-common-logic-0.0.1-SNAPSHOT.jar
cavali-fn-web-services-client-backend-logic-0.0.1-SNAPSHOT.jar
cavali-fn-web-services-common-backend-logic-0.0.1-SNAPSHOT.jar
cavali-fn-web-services-common-backend-rest-beans-0.0.1-SNAPSHOT.jar
cavali-fn-web-services-common-backend-ws-client-autogen-code-0.0.1-SNAPSHOT.jar
cavali-fn-web-services-core-backend-autogen-code-0.0.1-SNAPSHOT.jar
cavali-fn-web-services-core-backend-logic-0.0.1-SNAPSHOT.jar''', description: 'Los nombres de los jars a reemplazar en las respectivas librerias compartidas', trim: false), 
                booleanParam(defaultValue: true, description: 'indica si se realizaran los despliegues de las aplicaciones', name: 'REALIZAR_DESPLIEGUE_DE_APLICACIONES'),
				booleanParam(defaultValue: true, description: 'indica si se realizara un escaneo de codigo o no', name: 'REALIZAR_ESCANEO_DE_CODIGO'),
				booleanParam(defaultValue: true, description: 'indica si se debería terminar el despliegue si al terminar de procesar el umbral de calidad, este indicara un estado diferente a OK o verde', name: 'PARAR_EL_DESPLIEGUE_SI_EL_UMBRAL_FALLA'),
				booleanParam(defaultValue: true, description: 'indica si se debe cargar los ears y jars compilados al artifactory', name: 'CARGAR_ARTEFACTOS_AL_ARTIFACTORY'),
				booleanParam(defaultValue: true, description: 'indica si se debe cargar los ears y jars que figuran en el archivo de configuracion al artifactory, si esta activado, se cargaran todos los artefactos mapeados en las variables JARS_CONSTRUIBLES_A_LLAVES_DE_LIBRERIAS_COMPARTIDAS y NOMBRE_DE_APLICACION_A_CONFIGURACIONES_DE_DESPLIEGUE del archivo de configuracion, si esta desactivado, se cargara solamente los artefactos desplegados ', name: 'CARGAR_TODOS_LOS_ARTEFACTOS_EN_EL_ARCHIVO_DE_CONFIGURACION_AL_ARTIFACTORY'),
                string(name: 'EMPEZAR_DESDE_LA_TAREA_NUMERO', defaultValue: '1', description: 'Indica desde que numero de tarea empezar. La tarea de leer propiedades no se puede saltar ' + 
                             '1 Preparacion para escaneo, 2= Compilar con o sin escaneo, 3=Desinstalar applicaciones, 4=Reemplazar Jars, 5=Desplegar, 6=Versionar, 7=Limpiar', trim: false),
            ])
        ]
    )
    def tcl_empty_string = '\u007b\u007d'
    def jenkins_pipeline_script_prefix='@script'
    def build_workspace = "${env.WORKSPACE}${jenkins_pipeline_script_prefix}"
    def build_props_file_directory='app_build_files'
    def jars_to_replace_base_directory='jars_to_replace/'
    def ears_to_deploy_directory='ears_to_deploy/'
    def key_value_pair_groups_sep='||||'
    def key_value_pair_groups_split_regex='\\|\\|\\|\\|'
    def key_value_pair_sep='|||'
    def key_value_pair_split_regex='\\|\\|\\|'
    def key_value_sep='||'
    def key_value_split_regex='\\|\\|'
	def internal_config_key_value_pair_sep='``'
	def internal_config_key_value_sep='`'
    def stash_credentials_id = params.ID_DE_CREDENCIALES_DE_REPO_DE_PARCHE
    def stash_repo_path = params.RUTA_DE_REPO_DE_PARCHE
    def stash_branch = params.BRANCH_DEL_REPO_DE_PARCHE_A_USAR
    def configuration_file_path = params.RUTA_DEL_ARCHIVO_DE_CONFIGURACION_EN_EL_REPO_DE_PARCHE
    def build_props
    def git_source_repo_path
    def git_credentials_id
	def perform_deployment_of_apps = params.REALIZAR_DESPLIEGUE_DE_APLICACIONES
    def dmgr_server_key_to_configs = [:]
    def shared_library_key_to_configs = [:]
    def app_name_to_deploy_configs = [:]
    def app_name_to_shared_library_keys = [:]
    def buildable_jar_name_to_shared_library_keys = [:]
    def shared_library_server_key_to_configs = [:]
    def app_names_to_deploy = [:]
    def topology_key_to_configs = [:]
	def module_server_mapping_key_to_configs = [:]
	def role_to_user_and_or_group_mapping_key_to_configs = [:]
	def tuning_parameters_key_to_configs = [:]
	def cookie_config_key_to_configs = [:]
	def session_management_config_key_to_configs = [:]
    def jar_names_to_replace
	def interval_between_checks_of_app_status_to_start_in_milliseconds
    def amount_of_times_to_check_app_status_to_start 
	def perform_code_scan = params.REALIZAR_ESCANEO_DE_CODIGO
	def stop_build_on_quality_gate_failure = params.PARAR_EL_DESPLIEGUE_SI_EL_UMBRAL_FALLA
	def sonarqube_server_name
	def sq_project_key_prefix
	def sq_quality_profile_name_to_languages_for_project_creation
	def sp_quality_gate_name_for_project_creation
	def production_branch
	def initial_scan_type
    def jenkins_sonarqube_credential_id
	def maven_sonarqube_scan_plugin_name_and_version
	def amount_of_minutes_to_wait_for_scan_to_finish
	def git_repo_info
	def sq_project_info
	def upload_artifacts_to_artifactory = params.CARGAR_ARTEFACTOS_AL_ARTIFACTORY
	def upload_all_configured_artifacts_in_properties_file_to_artifactory = params.CARGAR_TODOS_LOS_ARTEFACTOS_EN_EL_ARCHIVO_DE_CONFIGURACION_AL_ARTIFACTORY
    def artifactory_server_id
	def artifactory_relative_directory_path_to_upload_ears
	def artifactory_relative_directory_path_to_upload_jars 
	def jdk_tool_name
    def maven_tool_name
    def maven_goals
    def maven_compilation_flags
    def maven_pom_file_path
	def start_from_task_number
    def current_task_number = 0;
	
	
	def initial_scan_types_property_file_to_enum = ['NINGUNO':INITIAL_SCAN_TYPES.NONE, 'SIN_CODIGO':INITIAL_SCAN_TYPES.NO_SOURCES, 'PRODUCCION':INITIAL_SCAN_TYPES.PRODUCTION]
	def initial_scan_types = 
    stage('Leer propiedades'){
      echo "params ${params}"
      dir (build_workspace){
        def git_remote_configs
        if (stash_credentials_id == null || stash_credentials_id.isEmpty()) {
          git_remote_configs = [url: stash_repo_path]
        } else {
          git_remote_configs = [url: stash_repo_path, credentialsId: stash_credentials_id]
        }
        checkout([$class: 'GitSCM', 
            branches: [[name: stash_branch]], 
            doGenerateSubmoduleConfigurations: false, 
            extensions: [
                [$class: 'CleanCheckout'],
                [$class: 'RelativeTargetDirectory', relativeTargetDir: build_props_file_directory]
            ], 
            submoduleCfg: [], 
            userRemoteConfigs: [
                git_remote_configs
            ]
        ])        
      }

      build_props = readProperties file: build_workspace + '/' + build_props_file_directory + '/' + configuration_file_path
	  
	  //perform_deployment_of_apps = ("true".equals(build_props.REALIZAR_DESPLIEGUE_DE_APLICACIONES))
      git_source_repo_path = build_props.RUTA_DE_REPOSITORIO_GIT;
      dmgr_server_key_to_configs = mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects (build_props.INFORMACION_DE_SERVIDORES_DMGR, 
          ['llave_servidor_dmgr':'dmgr_server_key', 'ip_servidor':'server_ip', 'puerto_ssh_servidor':'ssh_server_port', 'id_credenciales_servidor':'server_credentials_id',
		  'servidor_jenkins_y_servidor_dmgr_son_el_mismo':'jenkins_server_and_dmgr_server_are_the_same'], 'llave_servidor_dmgr', key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);
 
	  def dmgr_servers_paths_var_mapping = ['llave_servidor_dmgr': 'dmgr_server_key', 'dir_perfil_dmgr': 'dmgr_profile_path', 
		'dir_artefactos': 'dmgr_artifact_directory', 'dir_scripts_despliegue': 'dmgr_deploy_scripts_directory','ruta_cred_wsadmin': 'wsadmin_credential_properties_file_path' ]
	  
	  dmgr_servers_paths_info_str = build_props.INFORMACION_DE_RUTAS_DE_SERVIDORES_DMGR
      dmgr_servers_paths_info = dmgr_servers_paths_info_str.split(key_value_pair_groups_split_regex)
      dmgr_servers_paths_info.each{ dmgr_server_paths_info ->
        def dmgr_server_paths_config = dmgr_server_paths_info.split(key_value_pair_split_regex)
		def key_value_map = [:]
        for (int i = 0; i < dmgr_server_paths_config.length; i++) {
          def key_value_config = dmgr_server_paths_config[i].split(key_value_split_regex)
          def key_name = dmgr_servers_paths_var_mapping.get(key_value_config[0])
          if((key_value_config.length > 1) && key_name != null){
            key_value_map.put(key_name, key_value_config[1])
          }else{
            echo "no se considerara ${key} - ${value}";
          }
		}
        def dmgr_server_configs = dmgr_server_key_to_configs.get(key_value_map.get('dmgr_server_key'));
        dmgr_server_configs.each { dmgr_server_config ->
		  key_value_map.each { key, value -> dmgr_server_config.put(key, value)}
        }
      }

      shared_library_server_key_to_configs = mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects(build_props.INFORMACION_DE_SERVIDORES_DE_LIBRERIAS_COMPARTIDAS, 
          ['llave_servidor_libreria_compartida':'shared_library_server_key', 'ip_servidor':'server_ip', 'puerto_ssh_servidor':'ssh_server_port', 
          'id_credenciales_servidor':'server_credentials_id', 'dir_artefactos':'server_artifact_directory', 'servidor_jenkins_y_servidor_libreria_son_el_mismo': 'jenkins_server_and_library_server_are_the_same' ],
		  'llave_servidor_libreria_compartida', key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);

      shared_library_key_to_configs = mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects(build_props.INFORMACION_DE_LIBRERIAS_COMPARTIDAS, 
          ['llave_libreria':'shared_library_key', 'nombre_libreria':'shared_library_name', 'llave_servidor_dmgr':'dmgr_server_key', 
          'nombre_obj_contenedor':'container_obj_name', 'tipo_obj_contenedor':'container_obj_type', 
          'llave_servidor_libreria_compartida':'shared_library_server_key' ], 'llave_libreria',
          key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);

      buildable_jar_name_to_shared_library_keys = mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects(build_props.JARS_CONSTRUIBLES_A_LLAVES_DE_LIBRERIAS_COMPARTIDAS, 
          ['llave_libreria':'shared_library_key', 'nombre_jar':'jar_name'], 'nombre_jar',
          key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);
      
      topology_key_to_configs = mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects(build_props.INFORMACION_DE_TOPOLOGIAS_PARA_DESPLIEGUE, 
          ['llave_topologia':'topology_key', 'servidor_app':'app_server', 'nodo_app':'app_node', 'cluster_app':'app_cluster', 'celda_app':'app_cell', 
          'servidor_ihs':'ihs_server', 'nodo_ihs':'ihs_node', 'celda_ihs':'ihs_cell', ], 'llave_topologia',
          key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);

      module_server_mapping_key_to_configs = mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects(build_props.INFORMACION_DE_MAPEO_DE_MODULOS_A_SERVIDORES, 
          ['llave_mapeo_modulos_servidores':'module_server_mapping_key', 'artefacto_y_ruta_de_archivo_de_configuracion':'artifact_and_configuration_file_path', 
          'nombre_de_visualizacion_del_archivo_de_configuracion': 'config_file_display_name', 'llave_topologia':'topology_key'], 'llave_mapeo_modulos_servidores',
          key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);

      role_to_user_and_or_group_mapping_key_to_configs = mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects(build_props.INFORMACION_DE_MAPEO_DE_ROLES_DE_SEGURIDAD_A_USUARIOS_Y_O_GRUPOS,
          ['llave_mapeo_rol_a_usuarios_y_o_grupos':'role_to_user_and_or_group_mapping_key', 'nombre_de_rol':'role_name', 'permitir_acceso_a_todos':'allow_access_to_all', 'usuarios_a_mapear':'users_to_map', 'grupos_a_mapear':'groups_to_map',
          'permitir_acceso_a_usuarios_autenticados':'allow_access_to_authenticated_users', 'permitir_todos_los_usuarios_autenticados_en_los_dominios_de_la_aplicacion':'allow_all_authenticated_users_in_application_domain'], 'llave_mapeo_rol_a_usuarios_y_o_grupos', key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);

	  cookie_config_key_to_configs = mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects(build_props.INFORMACION_DE_CONFIGURACION_DE_COOKIES, 
          ['llave_configuracion_cookie':'cookie_config_key', 'nombre_de_cookie':'name', 'usar_https_en_cookie':'secure','usar_httponly_en_cookie':'httpOnly', 
		  'dominio_de_cookie':'domain', 'usar_raiz_de_contexto_como_ruta_en_cookie':'useContextRootAsPath', 'ruta_customizada_de_cookie':'path'], 
		  'llave_configuracion_cookie', key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);		  

	  tuning_parameters_key_to_configs = mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects(build_props.INFORMACION_DE_PARAMETROS_DE_AFINACION_DE_SESION, 
          ['llave_parametro_afinacion_sesion':'tuning_parameters_key', 'cantidad_maxima_de_sesiones':'maxInMemorySessionCount', 'permitir_mas_sesiones_que_la_cantidad_maxima':'allowOverflow',
		  'activar_timeout_de_sesion':'scheduleInvalidation', 'timeout_de_sesion_en_minutos':'invalidationTimeout'], 
		  'llave_parametro_afinacion_sesion', key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);
		  //llave_parametro_afinacion_sesion||parametro_sesion_client_web_app|||cantidad_maxima_de_sesiones||1000|||activar_timeout_de_sesion||true|||timeout_de_sesion_en_minutos||30
		  
      session_management_config_key_to_configs = mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects(build_props.INFORMACION_DE_CONFIGURACION_DE_GESTOR_DE_SESION, 
          ['llave_configuracion_gestor_sesion':'session_management_config_key', 'llave_configuracion_cookie':'cookie_config_key', 'llave_parametro_afinacion_sesion':'tuning_parameters_key', 
          'activar_rastreo_ssl':'enableSSLTracking', 'activar_cookies':'enableCookies' , 'permitir_reescritura_de_urls':'enableUrlRewriting', 'permitir_cambio_de_protocolo_en_reescritura':'enableProtocolSwitchRewriting',
		  'permitir_integracion_de_seguridad':'enableSecurityIntegration', 'permitir_acceso_serializado_a_sesiones':'allowSerializedSessionAccess', 'tiempo_maximo_de_espera_para_acceder_a_sesion_serializada':'maxWaitTime',
		  'modo_de_persistencia_para_sesiones':'sessionPersistenceMode', 'activar_gestor_de_sesiones':'enable'], 'llave_configuracion_gestor_sesion', key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);
		  
      app_name_to_deploy_configs = mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects(build_props.NOMBRE_DE_APLICACION_A_CONFIGURACIONES_DE_DESPLIEGUE, 
          ['nombre_app':'app_name', 'nombre_ear':'ear_name', 'llave_servidor_dmgr':'dmgr_server_key', 'llave_mapeo_modulos_servidores':'module_server_mapping_key',
		  'llave_mapeo_rol_a_usuarios_y_o_grupos':'role_to_user_and_or_group_mapping_key', 'llave_configuracion_gestor_sesion':'session_management_config_key'], 'nombre_app',
          key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);

      app_name_to_shared_library_keys = mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects(build_props.NOMBRE_DE_APLICACION_A_LIBRERIAS_COMPARTIDAS, 
          ['nombre_app':'app_name', 'llave_libreria':'shared_library_key'], 'nombre_app',
          key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);

      app_names_tp_deploy_array = params.NOMBRES_DE_APLICACIONES_A_DESPLEGAR.split('\n')
      app_names_to_deploy = []
      app_names_tp_deploy_array.each {app_names_to_deploy.add(it)}
      
      jars_to_replace_str = params.JARS_A_REEMPLAZAR
      jar_names_to_replace_array = jars_to_replace_str.split('\n')
      jar_names_to_replace = []
      jar_names_to_replace_array.each {jar_names_to_replace.add(it)}

      amount_of_times_to_check_app_status_to_start = build_props.CANTIDAD_DE_REVISIONES_DEL_ESTADO_DE_LA_APLICACION
      interval_between_checks_of_app_status_to_start_in_milliseconds = build_props.INTERVALO_DE_REVISION_DEL_ESTADO_DE_LA_APLICACION_EN_MILLISEGUNDOS
      start_from_task_number = params.EMPEZAR_DESDE_LA_TAREA_NUMERO.toInteger()
      //upload_artifacts_to_artifactory = ("true".equals(build_props.CARGAR_ARTEFACTOS_AL_ARTIFACTORY))
      artifactory_server_id = build_props.ID_DE_SERVIDOR_ARTIFACTORY
	  artifactory_relative_directory_path_to_upload_ears = build_props.RUTA_RELATIVA_DE_DIRECTORIO_ARTIFACTORY_PARA_CARGAR_EARS
	  artifactory_relative_directory_path_to_upload_jars = build_props.RUTA_RELATIVA_DE_DIRECTORIO_ARTIFACTORY_PARA_CARGAR_JARS
	  //perform_code_scan = ("true".equals(build_props.REALIZAR_ESCANEO_DE_CODIGO))
	  sonarqube_server_name = build_props.NOMBRE_DE_SERVIDOR_SONARQUBE
	  sq_project_key_prefix = build_props.PREFIJO_DE_LLAVE_DE_PROYECTO_SONARQUBE
	  
	  sq_quality_profile_name_to_languages_for_project_creation = 
	  mapKeyAndValuePairGroupsToKeyMappedToMultipleObjects(build_props.NOMBRE_DEL_PERFIL_DE_CALIDAD_A_LENGUAJE_A_USAR_PARA_CREAR_PROYECTO, 
          ['nombre_perfil_calidad':'quality_profile_name', 'lenguaje_a_usar':'language'], 'nombre_perfil_calidad',
          key_value_pair_groups_split_regex, key_value_pair_split_regex, key_value_split_regex);
		  
	  sp_quality_gate_name_for_project_creation = build_props.NOMBRE_DEL_UMBRAL_DE_CALIDAD_A_USAR_PARA_CREAR_PROYECTO
	  production_branch = build_props.NOMBRE_DE_LA_RAMA_DE_PRODUCCION
	  initial_scan_type = initial_scan_types_property_file_to_enum.get(build_props.TIPO_DE_ESCANEO_INICIAL)
	  initial_scan_type = (initial_scan_type) ? initial_scan_type : INITIAL_SCAN_TYPES.NONE;
	  jenkins_sonarqube_credential_id = build_props.ID_DE_CREDENCIAL_SONARQUBE
	  maven_sonarqube_scan_plugin_name_and_version = build_props.NOMBRE_Y_VERSION_DEL_PLUGIN_PARA_EL_ESCANEO_POR_MAVEN
	  amount_of_minutes_to_wait_for_scan_to_finish = (build_props.CANTIDAD_DE_MINUTOS_A_ESPERAR_A_QUE_TERMINE_EL_ESCANEO)?build_props.CANTIDAD_DE_MINUTOS_A_ESPERAR_A_QUE_TERMINE_EL_ESCANEO.toInteger():-1
	  //stop_build_on_quality_gate_failure =  ("true".equals(build_props.PARAR_EL_DESPLIEGUE_SI_EL_UMBRAL_FALLA))
	  jdk_tool_name=build_props.NOMBRE_DE_JDK_CONFIGURADO
      maven_tool_name=build_props.NOMBRE_DE_MAVEN_CONFIGURADO
	  maven_compilation_flags = build_props.MAVEN_INDICADORES_COMPILACION
      maven_goals = build_props.MAVEN_METAS_A_EJECUTAR
      maven_pom_file_path = build_props.MAVEN_RUTA_RELATIVA_ARCHIVO_POM
	  if (perform_code_scan){
	    def maven_pom_info = getProjectInfoFromPom(build_workspace, jdk_tool_name, maven_tool_name, maven_pom_file_path);
	    git_repo_info = getGitRepositoryInfo(build_workspace, production_branch)
	    sq_project_info = getSonarQubeProjectInfo (sonarqube_server_name, jenkins_sonarqube_credential_id, build_workspace, git_repo_info, maven_pom_info, sq_project_key_prefix, production_branch)
		echo "maven_pom_info ${maven_pom_info}"
	  }
	  echo "perform_deployment_of_apps ${perform_deployment_of_apps}"
      echo "git_source_repo_path ${git_source_repo_path}"
      echo "git_credentials_id ${git_credentials_id}"
      echo "dmgr_server_key_to_configs ${dmgr_server_key_to_configs}"
      echo "shared_library_key_to_configs ${shared_library_key_to_configs}"
	  echo "shared_library_server_key_to_configs ${shared_library_server_key_to_configs}"
      echo "buildable_jar_name_to_shared_library_keys ${buildable_jar_name_to_shared_library_keys}"
      echo "topology_key_to_configs ${topology_key_to_configs}"
      echo "module_server_mapping_key_to_configs ${module_server_mapping_key_to_configs}"
	  echo "role_to_user_and_or_group_mapping_key_to_configs ${role_to_user_and_or_group_mapping_key_to_configs}"
	  echo "tuning_parameters_key_to_configs ${tuning_parameters_key_to_configs}"
	  echo "cookie_config_key_to_configs ${cookie_config_key_to_configs}"
      echo "session_management_config_key_to_configs ${session_management_config_key_to_configs}"
      echo "app_name_to_deploy_configs ${app_name_to_deploy_configs}"
      echo "app_name_to_shared_library_keys ${app_name_to_shared_library_keys}"
      echo "app_names_to_deploy ${app_names_to_deploy}"
      echo "jar_names_to_replace ${jar_names_to_replace}"
      echo "amount_of_times_to_check_app_status_to_start ${amount_of_times_to_check_app_status_to_start }"
      echo "interval_between_checks_of_app_status_to_start_in_milliseconds ${interval_between_checks_of_app_status_to_start_in_milliseconds}"
      echo "start_from_task_number ${start_from_task_number}"
      echo "upload_artifacts_to_artifactory ${upload_artifacts_to_artifactory}"
      echo "artifactory_server_id ${artifactory_server_id}"
	  echo "artifactory_relative_directory_path_to_upload_ears ${artifactory_relative_directory_path_to_upload_ears}"
	  echo "artifactory_relative_directory_path_to_upload_jars ${artifactory_relative_directory_path_to_upload_jars}"
	  echo "perform_code_scan ${perform_code_scan}"
	  echo "sonarqube_server_name ${sonarqube_server_name}"
	  echo "sq_project_key_prefix ${sq_project_key_prefix}"
	  echo "sq_quality_profile_name_to_languages_for_project_creation ${sq_quality_profile_name_to_languages_for_project_creation}"
	  echo "sp_quality_gate_name_for_project_creation ${sp_quality_gate_name_for_project_creation}"
	  echo "production_branch ${production_branch}"
	  echo "initial_scan_type ${initial_scan_type}"
	  echo "jenkins_sonarqube_credential_id ${jenkins_sonarqube_credential_id}"
	  echo "maven_sonarqube_scan_plugin_name_and_version ${maven_sonarqube_scan_plugin_name_and_version}"
	  echo "amount_of_minutes_to_wait_for_scan_to_finish ${amount_of_minutes_to_wait_for_scan_to_finish}"
	  echo "git_repo_info ${git_repo_info}"
	  echo "sq_project_info ${sq_project_info}"
	  echo "stop_build_on_quality_gate_failure ${stop_build_on_quality_gate_failure}"
	  echo "jdk_tool_name ${jdk_tool_name}"
      echo "maven_tool_name ${maven_tool_name}"
      echo "maven_compilation_flags ${maven_compilation_flags}"
      echo "maven_goals ${maven_goals}"
      echo "maven_pom_file_path ${maven_pom_file_path}"
    }
	stage('Preparacion para escanear'){
      current_task_number++
      if(start_from_task_number > current_task_number){
          echo 'saltando tarea'
          return
      }
	  if (!perform_code_scan){
		echo 'el parametro REALIZAR_ESCANEO_DE_CODIGO esta desactivado, no se hara la preparacion preliminar para escanear el codigo'
        return
	  }
	  if (sq_project_info.projectExistsInSonarqube){
		echo "Proyecto ${sq_project_info.projectKey} ya existe, se procedera al siguiente paso"
		return;
	  }
      def scan_prep_info = scanPreparationStep (sonarqube_server_name, jenkins_sonarqube_credential_id, build_workspace, sq_project_info, sq_project_key_prefix, sq_quality_profile_name_to_languages_for_project_creation, sp_quality_gate_name_for_project_creation, production_branch)
	  if(scan_prep_info.error){
		if(scan_prep_info.projectCreated){
		  deleteSonarQubeProject(sonarqube_server_name, jenkins_sonarqube_credential_id, scan_prep_info, production_branch)
		}
		error ""
	  }
	  if (initial_scan_type.equals(INITIAL_SCAN_TYPES.NONE)){
		echo "se omitira el escaneo inicial y se procedera al siguiente paso"
		return;
	  }
	  if(scan_prep_info.projectCreated){
		echo "realizando escaneo preliminar"
		def sonar_pom_properties_override = [:]
		if (initial_scan_type.equals(INITIAL_SCAN_TYPES.PRODUCTION)){
		  dir (build_workspace){ sh "git checkout -f ${git_repo_info.productionBranch}" }
		} else if(initial_scan_type.equals(INITIAL_SCAN_TYPES.NO_SOURCES)){
			sonar_pom_properties_override.put('sonar.sources','')
		}
		codeScanStep(sonarqube_server_name, maven_pom_file_path, build_workspace, jdk_tool_name, maven_tool_name, maven_compilation_flags, maven_goals, maven_sonarqube_scan_plugin_name_and_version, 
		sq_project_info, git_repo_info, sonar_pom_properties_override, amount_of_minutes_to_wait_for_scan_to_finish, stop_build_on_quality_gate_failure)
		if ([INITIAL_SCAN_TYPES.NO_SOURCES,INITIAL_SCAN_TYPES.PRODUCTION].contains(initial_scan_type)){
		  dir (build_workspace){ sh "git checkout -f ${git_repo_info.branch}" }
		}
	  }
    }
    stage('Compilacion con o sin escaneo'){
      current_task_number++
      if(start_from_task_number > current_task_number){
        echo 'saltando tarea'
        return
      }
	  if (perform_code_scan){
		echo "realizando compilacion y escaneo de codigo"
		codeScanStep(sonarqube_server_name, maven_pom_file_path, build_workspace, jdk_tool_name, maven_tool_name, maven_compilation_flags, maven_goals, maven_sonarqube_scan_plugin_name_and_version, 
		sq_project_info, git_repo_info, null, amount_of_minutes_to_wait_for_scan_to_finish, stop_build_on_quality_gate_failure)
	  }else{
		echo 'el parametro REALIZAR_ESCANEO_DE_CODIGO esta desactivado, solo se realizara la compilacion de codigo'
	    compileStep(maven_pom_file_path, build_workspace, jdk_tool_name, maven_tool_name, maven_compilation_flags, maven_goals)
	  }
    }
    stage('Desinstalar aplicaciones'){
      current_task_number++
      if(start_from_task_number > current_task_number){
        echo 'saltando tarea'
        return
      }
	  if(!perform_deployment_of_apps){
        echo 'el parametro REALIZAR_DESPLIEGUE_DE_APLICACIONES esta desactivado, no se desintalaran las aplicaciones'
        return
      }
      uninstallApplicationsStep(app_name_to_deploy_configs, app_names_to_deploy, module_server_mapping_key_to_configs, topology_key_to_configs, dmgr_server_key_to_configs,
        key_value_sep, key_value_pair_sep, key_value_pair_groups_sep)
    }
    stage('Reemplazar jars'){
      current_task_number++
      if(start_from_task_number > current_task_number){
        echo 'saltando tarea'
        return
      }
	  if(!perform_deployment_of_apps){
		echo 'el parametro REALIZAR_DESPLIEGUE_DE_APLICACIONES esta desactivado, no se reemplazarn los jars'
        return
      }
      jarReplacementStep (buildable_jar_name_to_shared_library_keys, jar_names_to_replace, app_name_to_shared_library_keys, app_name_to_deploy_configs, app_names_to_deploy,
        shared_library_key_to_configs, shared_library_server_key_to_configs, build_workspace, dmgr_server_key_to_configs, module_server_mapping_key_to_configs, topology_key_to_configs, 
        jars_to_replace_base_directory, key_value_sep, key_value_pair_sep, key_value_pair_groups_sep)
    }
    stage('Desplegar aplicaciones'){
      current_task_number++
      if(start_from_task_number > current_task_number){
        echo 'saltando tarea'
        return
      }
	  if(!perform_deployment_of_apps){
        echo 'el parametro REALIZAR_DESPLIEGUE_DE_APLICACIONES esta desactivado, no se desplegaran las aplicaciones'
        return
      }
      deployApplicationsStep(app_name_to_deploy_configs, app_names_to_deploy, build_workspace, ears_to_deploy_directory, dmgr_server_key_to_configs, module_server_mapping_key_to_configs, 
	    topology_key_to_configs, role_to_user_and_or_group_mapping_key_to_configs, session_management_config_key_to_configs, tuning_parameters_key_to_configs, 
		cookie_config_key_to_configs, app_name_to_shared_library_keys, shared_library_key_to_configs, amount_of_times_to_check_app_status_to_start, 
		interval_between_checks_of_app_status_to_start_in_milliseconds, tcl_empty_string, key_value_sep, key_value_pair_sep, key_value_pair_groups_sep, 
        internal_config_key_value_sep, internal_config_key_value_pair_sep)
    }
    stage('Versionar con artifactory'){
      current_task_number++
      if(start_from_task_number > current_task_number){
        echo 'saltando tarea'
        return
      }
	  if(!upload_artifacts_to_artifactory){
		echo 'el parametro CARGAR_ARTEFACTOS_AL_ARTIFACTORY esta desactivado, no se subira artefactos al artifactory'
        return
      }
      artifactoryUploadStep(build_workspace, artifactory_server_id, artifactory_relative_directory_path_to_upload_ears, app_name_to_deploy_configs, app_names_to_deploy, 
        buildable_jar_name_to_shared_library_keys, jar_names_to_replace, artifactory_relative_directory_path_to_upload_jars, upload_all_configured_artifacts_in_properties_file_to_artifactory)
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
