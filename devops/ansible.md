# Ansible tips by Frodo
#### Ad-hoc command for debugging host variables
````
ansible all -i inventory/testing/hosts -m debug -a "msg='{{ hostvars[inventory_hostname] }}'" -l kafka-broker-01.cluster.company.dev
````
#### Insert a task that will stop playbook/role immediatly, useful for debugging
````
    - name: Debug
      debug:
        msg: 
          -  "{{ some_variable_you_need_to_debug_before_playing_further }}"

    # This is the task
    - name: Quit immediately
      meta: end_play
````
