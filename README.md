##### vue组件 传递

- org为最外层组件，他有三个子组件orgChildren、orgUsers、orgTree
- orgUsers 下还引用了 simpleSelectUser.vue组件
- 我在写simpleSelectUser.vue组件的时候需要点击orgTree的orgId
- orgTree向org 传（在orgTree中叫rootId）
```
props: {
    rootId: {
            type: String
    },
    eventBus: {
            type: Object
        },
}

watch: {
        // 如果 rootId 发生改变，这个函数就会运行
        rootId: function (newRootId) {
            this.reload();
        }
    },
```
- 在orgTree中会通过点击改变子节点
···
        //树节点点击事件
        handleNodeClick(nodeData) {
            // console.log('treeNodeClicked: ' + nodeData.ID + ', compId: ' + this.compId);
            this.selectedNodeData = nodeData;
            this.eventBus.$emit('org-tree-clicked', nodeData);
        },
···

##### 在org组件里

```
<org-tree :compId='orgTreeId' :eventBus='eventBus' :rootId='orgId' :customProps='orgTreeProps'></org-tree>

<org-users :compId='orgUserId' :eventBus='eventBus' :orgId='currOrgId' :customProps='orgUsersProps'></org-users>

<org-children :compId='orgChildrenId' :eventBus='eventBus' :orgId='orgId' :customProps='orgChildrenProps'></org-children>

data() {
        return {
            //事件总线，这里使用当前实例作为EventBus，而不是使用自定义EventBus
            eventBus: this,
            //根节点标识
            orgId: null,
            //当前选中树节点ID
            currOrgId: null,
        }
}
    mounted: function () {
        let vm = this;
        this.eventBus.$on('org-tree-clicked', function (node) {
            console.log("org: on org-tree-clicked event: " + node.ID);
            this.currOrgId = node.ID;
        });
    },

    components: {
        'org-tree': orgTree, 'org-users': orgUsers, 'org-children': orgChildren
    }
```

- 在orgUsers里
```
        eventBus: {
            type: Object
        },
        orgId: {
            type: String
        },

            mounted: function () {
        var vm = this;
        this.apiUrl = this.getBaseUrl();
        this.findUserList(this.tmpId, this.currentPage - 1, this.pageSize);

        //监听到点击事件后 刷新数据
        // this.eventBus.$on('org-tree-clicked', function (node) {
        //     this.tmpId = node.ID;
        //     vm.findUserList(this.tmpId, this.currentPage - 1, this.pageSize)
        // });
        //监听到事件后 将选择的用户提交上去
        this.eventBus.$on('simple-select-user-ok', function (value) {
            vm.addUser(value)
        });
    },
```
#### 在simpleSelectUser里
```
        eventBus: {
            type: Object
        },
        orgId:{
            type:String
        }
                //选择完成，通过事件将选择结果反馈回去
        onOK(value) {
            this.eventBus.$emit('simple-select-user-ok', value);
        },
```