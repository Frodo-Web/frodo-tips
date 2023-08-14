# Ansible tips by Frodo

#### Insert a task that will stop playbook/role immediatly, useful for debugging
````
    - name: Debug
      debug:
        msg: 
          -  "{{ some_variable_you_need_to_debug_before_playing_further }}"
    - name: Quit immediately
      meta: end_play
````
