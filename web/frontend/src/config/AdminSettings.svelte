<script>
    import { Row, Col } from 'sveltestrap'
    import { onMount } from 'svelte'
    import EditRole from './admin/EditRole.svelte'
    import AddUser from './admin/AddUser.svelte'
    import ShowUsers from './admin/ShowUsers.svelte'
    import Options from './admin/Options.svelte'

    let users = []

    function getUserList() {
        fetch('/api/users/?via-ldap=false&not-just-user=true')
            .then(res => res.json())
            .then(usersRaw => {
                users = usersRaw
        })
    }

    onMount(() => getUserList())

</script>

<Row cols={2} class="p-2 g-2" >
    <Col class="mb-1">
        <AddUser on:reload={getUserList}/>
    </Col>
    <Col class="mb-1">
        <ShowUsers on:reload={getUserList} bind:users={users}/>
    </Col>
    <Col>
        <EditRole on:reload={getUserList}/>
    </Col>
    <Col>
        <Options/>
    </Col>
</Row>
