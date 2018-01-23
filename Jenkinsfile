node {
    stage("Project refresh") {
        echo "Refreshing project in Ansible Tower."
        tower-cli project update -n "Tomcat Playbooks" --monitor
    }
    stage("Syntax check") {
        echo "Syntax checking playbooks"
        TEMPLATES=$(ls|grep yml|cut -d'.' -f1)
        # For all playbooks files we find in the cloned git repository
        for item in ${TEMPLATES[@]}; do
            # Run a syntax check for all the playbooks
            ansible-playbook --syntax-check ./$item.yml
        done
    }
    stage("Test run") {
        for item in ${TEMPLATES[@]}; do
            echo "Test run: $item"
        done
    }
    stage("Create job template") {
        for item in ${TEMPLATES[@]}; do
            echo "Create job template: $item"
        done
    }
}
