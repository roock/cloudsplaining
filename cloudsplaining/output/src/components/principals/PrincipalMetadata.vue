<template>
    <b-container>
        <dl class="row">
            <dt class="col-sm-5">ARN</dt>
            <dd class="col-sm-7 text-monospace"><small>{{getPrincipalMetadata(principalId, principalType)['arn']}}</small></dd>

            <dt class="col-sm-5">ID</dt>
            <dd class="col-sm-7 text-monospace">{{getPrincipalMetadata(principalId, principalType)['id']}}</dd>

            <dt class="col-sm-5">Excluded from scan</dt>
            <dd class="col-sm-7 text-monospace">{{getPrincipalMetadata(principalId, principalType)['is_excluded']}}</dd>

            <dt class="col-sm-5">Created</dt>
            <dd class="col-sm-7">{{getPrincipalMetadata(principalId, principalType)['create_date']}}</dd>

            <dt class="col-sm-5">Inline Policies</dt>
            <dd class="col-sm-7">
                <b-button size="sm"
                          v-b-toggle="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.inline-policies.collapse`">
                    {{ getPrincipalPolicyNames(principalId, principalType, "Inline").length }}
                </b-button>
                <b-collapse
                        v-bind:id="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.inline-policies.collapse`">
                    <br>
                    <pre><code>{{ getPrincipalPolicyNames(principalId, principalType, "Inline") }}</code></pre>
                </b-collapse>
            </dd>

            <dt class="col-sm-5">AWS-Managed Policies</dt>
            <dd class="col-sm-7">
                <b-button size="sm"
                          v-b-toggle="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.aws-managed-policies.collapse`">
                    {{ getPrincipalPolicyNames(principalId, principalType, "AWS").length }}
                </b-button>
                <b-collapse
                        v-bind:id="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.aws-managed-policies.collapse`">
                    <br>
                    <pre><code>{{ getPrincipalPolicyNames(principalId, principalType, "AWS") }}</code></pre>
                </b-collapse>
            </dd>

            <dt class="col-sm-5">Customer-Managed Policies</dt>
            <dd class="col-sm-7">
                <b-button size="sm"
                          v-b-toggle="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.customer-managed-policies.collapse`">
                    {{ getPrincipalPolicyNames(principalId, principalType, "Customer").length }}
                </b-button>
                <b-collapse
                        v-bind:id="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.customer-managed-policies.collapse`">
                    <br>
                    <pre><code>{{ getPrincipalPolicyNames(principalId, principalType, "Customer") }}</code></pre>
                </b-collapse>
            </dd>

            <!--ROLE SPECIFIC-->
            <template v-if="principalType.toLowerCase() === 'role'">
                <dt class="col-sm-5">Role Trust Policy</dt>
                <dd class="col-sm-7">
                    <b-button size="sm"
                              v-b-toggle="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.role-trust-policy.collapse`">
                        Details
                    </b-button>
                    <b-collapse
                            v-bind:id="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.role-trust-policy.collapse`">
                        <br>
                        <pre><code>{{ JSON.parse(JSON.stringify(getRoleTrustPolicy(principalId))) }}</code></pre>
                    </b-collapse>
                </dd>
                <!--Instance Profiles-->
                <dt class="col-sm-5">Instance Profiles</dt>
                <dd class="col-sm-7">
                    <b-button size="sm"
                              v-b-toggle="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.instance-profiles.collapse`">
                        {{ getPrincipalMetadata(principalId, principalType)['instance_profiles'].length }}
                    </b-button>
                    <b-collapse
                            v-bind:id="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.instance-profiles.collapse`">
                        <br>
                        <pre><code>{{ JSON.parse(JSON.stringify(getPrincipalMetadata(principalId, principalType)['instance_profiles'])) }}</code></pre>
                    </b-collapse>
                </dd>
            </template><!--/role specific details-->

            <!--GROUP SPECIFIC-->
            <template v-if="principalType.toLowerCase() === 'group'">
                <dt class="col-sm-3">Group Members</dt>
                <dd class="col-sm-9">
                    <b-button size="sm"
                            v-b-toggle="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.group-members.collapse`">
                        {{ getGroupMembers(principalId).length }}
                    </b-button>
                    <b-collapse
                            v-bind:id="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.group-members.collapse`">
                        Group Members:
                        <ul v-bind:key="groupMemberEntry" v-for="groupMemberEntry in getGroupMembers(principalId).length">
                          <li>{{ getGroupMembers(principalId)[groupMemberEntry - 1]['user_name'] }} (ID: {{ getGroupMembers(principalId)[groupMemberEntry - 1]['user_id'] }})</li>
                        </ul>
                    </b-collapse>
                </dd>
            </template>

            <!--USER SPECIFIC-->
            <template v-if="principalType.toLowerCase() === 'user'">
                <dt class="col-sm-3">Group Memberships</dt>
                <dd class="col-sm-9">
                    <b-button size="sm"
                              v-b-toggle="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.group-membership.collapse`">
                        {{ Object.keys(getGroupMemberships(principalId)).length }}
                    </b-button>
                    <b-collapse
                            v-bind:id="`iam.${principalType}.${getPrincipalMetadata(principalId, principalType)['id']}.group-membership.collapse`"
                            v-if="Object.keys(getGroupMemberships(principalId)).length === 0"
                    >
                        Group Memberships:
                        <ul v-bind:key="groupMembershipEntry" v-for="groupMembershipEntry in getGroupMemberships(principalId).length">
                          <li>{{ getGroupMemberships(principalId)[groupMembershipEntry - 1]['group_name'] }} (ID: {{ getGroupMemberships(principalId)[groupMembershipEntry - 1]['group_id'] }})</li>
                        </ul>
                    </b-collapse>
                </dd>
            </template>
        </dl>
    </b-container>

</template>

<script>
    const principalsUtil = require('../../util/principals');
    const rolesUtil = require('../../util/roles');
    const groupsUtil = require('../../util/groups');

    export default {
        name: "PrincipalMetadata",
        props: {
            iam_data: {
                type: Object
            },
            principalType: {
                type: String
            },
            principalId: {
                type: String
            }
        },
        methods: {
            getPrincipalMetadata: function (principalName, principalType) {
                return principalsUtil.getPrincipalMetadata(this.iam_data, principalName, principalType)
            },
            getPrincipalPolicyNames: function (principalName, principalType, policyType) {
                return principalsUtil.getPrincipalPolicyNames(this.iam_data, principalName, principalType, policyType)
            },
            getRoleTrustPolicy: function (roleId) {
                return rolesUtil.getTrustPolicyDocumentForRole(this.iam_data, roleId)
            },
            getGroupMembers: function (principalId) {
                return groupsUtil.getGroupMembers(this.iam_data, principalId)
            },
            getGroupMemberships: function (principalId) {
                return groupsUtil.getGroupMemberships(this.iam_data, principalId)
            }
        }
    }
</script>

<style scoped>

</style>
